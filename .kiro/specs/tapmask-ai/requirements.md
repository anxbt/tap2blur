# Requirements Document

## Introduction

TapMask AI is a serverless, cloud-native video privacy and editing platform designed for the "AI for Media & Content" hackathon track. The system enables users to upload videos and perform professional-grade object tracking and processing through a single-click interface. The platform leverages Meta's SAM 3 (Segment Anything Model 3) for segmentation and ProPainter for inpainting, providing two core modes: Privacy Blur for face/object anonymization and Magic Eraser for content removal. The architecture is built on AWS services with a focus on cost optimization through Spot Instances, achieving up to 70% cost reduction while maintaining scalability and performance.

## Glossary

- **TapMask_System**: The complete serverless video processing platform including frontend, API, orchestration, and compute components
- **User**: An individual who uploads and processes videos through the platform
- **Video_Upload**: A video file submitted by a User for processing
- **Segmentation_Point**: A single click coordinate provided by a User to identify an object or face for tracking
- **SAM3_Engine**: Meta's Segment Anything Model 3 used for object segmentation and tracking
- **ProPainter_Engine**: The inpainting model used for content removal in Magic Eraser mode
- **Privacy_Blur_Mode**: Processing mode that applies Gaussian blur to tracked objects for anonymization
- **Magic_Eraser_Mode**: Processing mode that removes tracked objects using inpainting
- **Processing_Job**: A queued task representing a video processing request
- **GPU_Worker**: An EC2 G5 Spot Instance with NVIDIA A10G GPU that executes processing jobs
- **Step_Function_Pipeline**: AWS Step Functions orchestration managing the processing workflow
- **Job_Queue**: Amazon ElastiCache (Redis) instance managing processing job state
- **Output_Video**: The processed video file ready for download
- **Free_Tier_User**: A User with access to 3 video processing jobs per month
- **Pro_Tier_User**: A User with unlimited video processing and 4K export capabilities

## Requirements

### Requirement 1: Video Upload and Storage

**User Story:** As a User, I want to upload videos securely, so that I can process them for privacy protection or content removal.

#### Acceptance Criteria

1. WHEN a User uploads a video file, THE TapMask_System SHALL validate the file format against supported formats (MP4, MOV, AVI, WebM)
2. WHEN a User uploads a video file, THE TapMask_System SHALL validate the file size does not exceed 500MB for Free_Tier_User
3. WHEN a User uploads a video file, THE TapMask_System SHALL validate the file size does not exceed 2GB for Pro_Tier_User
4. WHEN a valid video file is uploaded, THE TapMask_System SHALL store it in S3 Standard storage with AES-256 encryption
5. WHEN a video file is stored, THE TapMask_System SHALL generate a unique identifier for the Video_Upload
6. WHEN a video file is stored, THE TapMask_System SHALL return the unique identifier to the User within 5 seconds
7. WHEN an invalid video file is uploaded, THE TapMask_System SHALL return a descriptive error message to the User

### Requirement 2: Interactive Segmentation Interface

**User Story:** As a User, I want to click on a face or object in my video once, so that the system can automatically track it throughout the video.

#### Acceptance Criteria

1. WHEN a User views an uploaded video, THE TapMask_System SHALL display an interactive video player using Plyr.js
2. WHEN a User clicks on a point in the video frame, THE TapMask_System SHALL capture the Segmentation_Point coordinates
3. WHEN a Segmentation_Point is captured, THE TapMask_System SHALL display visual feedback within 200 milliseconds
4. WHEN a User provides a Segmentation_Point, THE TapMask_System SHALL allow the User to select Privacy_Blur_Mode or Magic_Eraser_Mode
5. WHEN a User confirms the processing mode, THE TapMask_System SHALL create a Processing_Job with the Video_Upload identifier, Segmentation_Point, and selected mode
6. WHEN multiple Segmentation_Points are provided for a single video, THE TapMask_System SHALL track all specified objects independently

### Requirement 3: Object Tracking with SAM3

**User Story:** As a User, I want the system to automatically track my selected object across all video frames, so that I don't have to manually mark each frame.

#### Acceptance Criteria

1. WHEN a Processing_Job is initiated, THE SAM3_Engine SHALL extract frames from the Video_Upload at 30 frames per second
2. WHEN frames are extracted, THE SAM3_Engine SHALL apply segmentation to the Segmentation_Point in the first frame
3. WHEN segmentation is applied, THE SAM3_Engine SHALL generate a binary mask for the identified object
4. WHEN a binary mask is generated, THE SAM3_Engine SHALL propagate the mask across all subsequent frames using temporal tracking
5. WHEN tracking fails on a frame, THE SAM3_Engine SHALL log the frame number and continue processing remaining frames
6. WHEN all frames are processed, THE SAM3_Engine SHALL output a mask sequence file containing all binary masks

### Requirement 4: Privacy Blur Processing

**User Story:** As a User, I want to blur faces or objects in my video, so that I can protect privacy while maintaining video context.

#### Acceptance Criteria

1. WHEN a Processing_Job specifies Privacy_Blur_Mode, THE TapMask_System SHALL apply OpenCV Gaussian blur to masked regions
2. WHEN applying Gaussian blur, THE TapMask_System SHALL use a kernel size of 99x99 pixels for effective anonymization
3. WHEN blur is applied to a frame, THE TapMask_System SHALL preserve all unmasked regions without modification
4. WHEN all frames are blurred, THE TapMask_System SHALL encode the frames into an Output_Video
5. WHEN encoding is complete, THE TapMask_System SHALL maintain the original video resolution and frame rate

### Requirement 5: Magic Eraser Processing

**User Story:** As a User, I want to remove objects from my video seamlessly, so that the video appears as if the object was never present.

#### Acceptance Criteria

1. WHEN a Processing_Job specifies Magic_Eraser_Mode, THE ProPainter_Engine SHALL apply inpainting to masked regions
2. WHEN applying inpainting, THE ProPainter_Engine SHALL use temporal coherence to ensure smooth transitions between frames
3. WHEN inpainting is applied to a frame, THE ProPainter_Engine SHALL fill masked regions with contextually appropriate content
4. WHEN all frames are inpainted, THE TapMask_System SHALL encode the frames into an Output_Video
5. WHEN encoding is complete, THE TapMask_System SHALL maintain the original video resolution and frame rate

### Requirement 6: Processing Pipeline Orchestration

**User Story:** As a system operator, I want the processing pipeline to be reliable and traceable, so that I can monitor job progress and handle failures gracefully.

#### Acceptance Criteria

1. WHEN a Processing_Job is created, THE Step_Function_Pipeline SHALL validate the job parameters
2. WHEN job parameters are valid, THE Step_Function_Pipeline SHALL transition the job through states: Validate, Extract, Track, Process, Encode
3. WHEN a pipeline state fails, THE Step_Function_Pipeline SHALL retry the state up to 3 times with exponential backoff
4. WHEN all retries are exhausted, THE Step_Function_Pipeline SHALL mark the job as failed and notify the User
5. WHEN a pipeline state succeeds, THE Step_Function_Pipeline SHALL update the Job_Queue with the current state
6. WHEN the Encode state completes, THE Step_Function_Pipeline SHALL mark the job as complete and store the Output_Video in S3

### Requirement 7: GPU Compute Resource Management

**User Story:** As a system operator, I want to optimize compute costs while maintaining performance, so that the platform remains economically viable.

#### Acceptance Criteria

1. WHEN processing demand increases, THE TapMask_System SHALL launch GPU_Worker instances using EC2 G5 Spot Instances
2. WHEN a GPU_Worker is launched, THE TapMask_System SHALL configure it with NVIDIA A10G GPU drivers and required ML libraries
3. WHEN a GPU_Worker is available, THE TapMask_System SHALL poll the Job_Queue for pending Processing_Jobs
4. WHEN a GPU_Worker retrieves a Processing_Job, THE TapMask_System SHALL execute the SAM3_Engine and ProPainter_Engine on the worker
5. WHEN processing demand decreases, THE TapMask_System SHALL terminate idle GPU_Worker instances after 5 minutes of inactivity
6. WHEN a Spot Instance interruption occurs, THE TapMask_System SHALL requeue the Processing_Job and assign it to another GPU_Worker

### Requirement 8: Content Delivery and Download

**User Story:** As a User, I want to download my processed video quickly, so that I can use it immediately after processing completes.

#### Acceptance Criteria

1. WHEN an Output_Video is stored in S3, THE TapMask_System SHALL generate a CloudFront signed URL with 24-hour expiration
2. WHEN a User requests download, THE TapMask_System SHALL serve the Output_Video through CloudFront CDN
3. WHEN serving through CloudFront, THE TapMask_System SHALL achieve download latency of less than 2 seconds for the first byte
4. WHEN a User downloads an Output_Video, THE TapMask_System SHALL track the download event for analytics
5. WHEN 24 hours elapse after Output_Video creation, THE TapMask_System SHALL delete the file using S3 Lifecycle policies

### Requirement 9: Data Security and Encryption

**User Story:** As a User, I want my videos to be encrypted and secure, so that my content remains private and protected.

#### Acceptance Criteria

1. WHEN a video file is uploaded, THE TapMask_System SHALL encrypt it using AES-256 encryption with AWS KMS
2. WHEN a video file is stored in S3, THE TapMask_System SHALL enable server-side encryption at rest
3. WHEN a video file is transmitted, THE TapMask_System SHALL use TLS 1.3 for encryption in transit
4. WHEN a User requests download, THE TapMask_System SHALL generate signed URLs with time-limited access
5. WHEN a video file is deleted, THE TapMask_System SHALL ensure cryptographic erasure of encryption keys

### Requirement 10: Scalability and Auto Scaling

**User Story:** As a system operator, I want the platform to scale automatically, so that it can handle varying loads from 1 to 10,000 concurrent users.

#### Acceptance Criteria

1. WHEN concurrent user count increases, THE TapMask_System SHALL scale API Gateway Lambda functions automatically
2. WHEN Job_Queue depth exceeds 10 pending jobs, THE TapMask_System SHALL launch additional GPU_Worker instances
3. WHEN Job_Queue depth falls below 5 pending jobs, THE TapMask_System SHALL terminate excess GPU_Worker instances
4. WHEN scaling GPU_Worker instances, THE TapMask_System SHALL maintain a minimum of 1 instance and maximum of 50 instances
5. WHEN API request rate exceeds 1000 requests per second, THE TapMask_System SHALL maintain response times under 500 milliseconds

### Requirement 11: User Tier Management and Quotas

**User Story:** As a User, I want to understand my usage limits, so that I can decide whether to upgrade to Pro tier.

#### Acceptance Criteria

1. WHEN a Free_Tier_User creates a Processing_Job, THE TapMask_System SHALL check the monthly job count
2. WHEN a Free_Tier_User has processed 3 videos in the current month, THE TapMask_System SHALL reject additional job requests with a quota exceeded message
3. WHEN a Pro_Tier_User creates a Processing_Job, THE TapMask_System SHALL allow unlimited job creation
4. WHERE a User is Pro_Tier_User, THE TapMask_System SHALL enable 4K resolution export option
5. WHEN a calendar month ends, THE TapMask_System SHALL reset the job count for all Free_Tier_User accounts

### Requirement 12: GDPR Compliance and Data Retention

**User Story:** As a User in the EU, I want my data to be handled according to GDPR regulations, so that my privacy rights are protected.

#### Acceptance Criteria

1. WHEN a User uploads a video, THE TapMask_System SHALL store data in EU regions if the User is located in the EU
2. WHEN a User requests data deletion, THE TapMask_System SHALL delete all associated Video_Upload and Output_Video files within 24 hours
3. WHEN a User requests data export, THE TapMask_System SHALL provide all stored data in machine-readable format within 30 days
4. WHEN 24 hours elapse after video upload, THE TapMask_System SHALL automatically delete the Video_Upload and Output_Video using S3 Lifecycle policies
5. WHEN processing personal data, THE TapMask_System SHALL log all data access events for audit purposes

### Requirement 13: Error Handling and User Notifications

**User Story:** As a User, I want to be notified of processing status and errors, so that I understand what is happening with my video.

#### Acceptance Criteria

1. WHEN a Processing_Job is created, THE TapMask_System SHALL send a job started notification to the User
2. WHEN a Processing_Job completes successfully, THE TapMask_System SHALL send a completion notification with download link
3. WHEN a Processing_Job fails, THE TapMask_System SHALL send an error notification with a descriptive error message
4. WHEN a Spot Instance interruption occurs, THE TapMask_System SHALL send a processing delayed notification to the User
5. WHEN a User's quota is exceeded, THE TapMask_System SHALL send a notification with upgrade options

### Requirement 14: API Gateway and Authentication

**User Story:** As a User, I want to authenticate securely, so that only I can access my processed videos.

#### Acceptance Criteria

1. WHEN a User accesses the API, THE TapMask_System SHALL require authentication via JWT tokens
2. WHEN a User authenticates, THE TapMask_System SHALL validate the JWT token signature and expiration
3. WHEN an invalid token is provided, THE TapMask_System SHALL return a 401 Unauthorized error
4. WHEN a User requests a resource, THE TapMask_System SHALL verify the User owns the requested resource
5. WHEN a User attempts to access another User's resource, THE TapMask_System SHALL return a 403 Forbidden error

### Requirement 15: Monitoring and Observability

**User Story:** As a system operator, I want to monitor system health and performance, so that I can identify and resolve issues proactively.

#### Acceptance Criteria

1. WHEN a Processing_Job is executed, THE TapMask_System SHALL log execution time for each pipeline state
2. WHEN a GPU_Worker processes a job, THE TapMask_System SHALL emit GPU utilization metrics to CloudWatch
3. WHEN an error occurs, THE TapMask_System SHALL log the error with stack trace and context to CloudWatch Logs
4. WHEN API latency exceeds 1 second, THE TapMask_System SHALL emit a high latency alarm
5. WHEN Spot Instance interruption rate exceeds 20 percent, THE TapMask_System SHALL emit a capacity alarm
