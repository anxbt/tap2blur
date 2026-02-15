# Implementation Plan: TapMask AI

## Overview

This implementation plan breaks down the TapMask AI platform into discrete coding tasks. The platform is built using Python for backend services, React with TypeScript for the frontend, and AWS CDK for infrastructure. The implementation follows a bottom-up approach, starting with core data models and utilities, then building the processing pipeline, API layer, and finally the frontend interface.

## Tasks

- [ ] 1. Setup project structure and core infrastructure
  - Create Python project with Poetry for dependency management
  - Setup AWS CDK project for infrastructure as code
  - Configure S3 buckets (uploads, outputs, frontend) with encryption
  - Configure ElastiCache Redis cluster with Multi-AZ
  - Setup CloudWatch log groups and metric namespaces
  - _Requirements: 1.4, 9.1, 9.2_

- [ ] 2. Implement data models and validation
  - [ ] 2.1 Create Pydantic models for User, Video, Job, and SegmentationMask
    - Define all fields with proper types and validation rules
    - Implement serialization methods (to_dict, from_dict)
    - _Requirements: 1.5, 2.5_
  
  - [ ]* 2.2 Write property test for unique video ID generation
    - **Property 5: Unique Video Identifiers**
    - **Validates: Requirements 1.5**
  
  - [ ] 2.3 Implement file upload validation logic
    - Create validate_upload function checking format and size
    - Implement tier-specific size limit checks
    - _Requirements: 1.1, 1.2, 1.3, 1.7_
  
  - [ ]* 2.4 Write property tests for upload validation
    - **Property 1: Valid File Format Acceptance**
    - **Property 2: Free Tier File Size Limit**
    - **Property 3: Pro Tier File Size Limit**
    - **Validates: Requirements 1.1, 1.2, 1.3**


- [ ] 3. Implement authentication and authorization
  - [ ] 3.1 Create JWT token validation middleware
    - Implement token signature verification
    - Implement token expiration checking
    - _Requirements: 14.1, 14.2, 14.3_
  
  - [ ]* 3.2 Write property tests for JWT validation
    - **Property 42: Authentication Requirement**
    - **Property 43: JWT Token Validation**
    - **Validates: Requirements 14.1, 14.2, 14.3**
  
  - [ ] 3.3 Implement resource ownership authorization
    - Create check_resource_ownership function
    - Implement 403 Forbidden error responses
    - _Requirements: 14.4, 14.5_
  
  - [ ]* 3.4 Write property test for resource ownership
    - **Property 44: Resource Ownership Authorization**
    - **Validates: Requirements 14.4, 14.5**

- [ ] 4. Implement user quota management
  - [ ] 4.1 Create quota checking and enforcement logic
    - Implement check_user_quota function for free tier
    - Implement monthly quota reset logic
    - Store quota data in Redis
    - _Requirements: 11.1, 11.2, 11.5_
  
  - [ ]* 4.2 Write property tests for quota management
    - **Property 29: Free Tier Quota Checking**
    - **Property 30: Free Tier Quota Enforcement**
    - **Property 31: Pro Tier Unlimited Jobs**
    - **Property 33: Monthly Quota Reset**
    - **Validates: Requirements 11.1, 11.2, 11.3, 11.5**
  
  - [ ] 4.3 Implement Pro tier feature flags
    - Create check_tier_features function
    - Enable 4K export for Pro tier users
    - _Requirements: 11.4_
  
  - [ ]* 4.4 Write property test for Pro tier features
    - **Property 32: Pro Tier 4K Export Availability**
    - **Validates: Requirements 11.4**

- [ ] 5. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 6. Implement S3 storage operations
  - [ ] 6.1 Create S3 upload handler with encryption
    - Implement upload_to_s3 function with KMS encryption
    - Generate unique S3 keys for each upload
    - Set proper metadata and content types
    - _Requirements: 1.4, 9.1, 9.2_
  
  - [ ]* 6.2 Write property test for upload encryption
    - **Property 4: Upload Encryption**
    - **Validates: Requirements 1.4, 9.1, 9.2**
  
  - [ ] 6.3 Implement CloudFront signed URL generation
    - Create generate_signed_url function with 24-hour expiration
    - Configure CloudFront key pair for signing
    - _Requirements: 8.1, 9.4_
  
  - [ ]* 6.4 Write property tests for signed URLs
    - **Property 22: Signed URL Generation with Expiration**
    - **Property 24: Signed URL Time-Limited Access**
    - **Validates: Requirements 8.1, 9.4**
  
  - [ ] 6.5 Implement S3 lifecycle policies
    - Configure 24-hour auto-deletion for uploads and outputs
    - _Requirements: 8.5, 12.4_

- [ ] 7. Implement Redis job queue operations
  - [ ] 7.1 Create Redis client wrapper
    - Implement connection pooling and retry logic
    - Create helper functions for job queue operations
    - _Requirements: 6.5_
  
  - [ ] 7.2 Implement job queueing and status tracking
    - Create enqueue_job function
    - Create update_job_status function
    - Create get_job_status function
    - _Requirements: 2.5, 6.5_
  
  - [ ]* 7.3 Write property test for job creation
    - **Property 8: Job Creation with Valid Parameters**
    - **Validates: Requirements 2.5**

- [ ] 8. Implement notification system
  - [ ] 8.1 Create notification sender using AWS SNS/SES
    - Implement send_notification function
    - Create email templates for each notification type
    - _Requirements: 13.1, 13.2, 13.3, 13.4, 13.5_
  
  - [ ]* 8.2 Write property tests for notifications
    - **Property 37: Job Started Notification**
    - **Property 38: Job Completion Notification**
    - **Property 39: Job Failure Notification**
    - **Property 40: Spot Interruption Notification**
    - **Property 41: Quota Exceeded Notification**
    - **Validates: Requirements 13.1, 13.2, 13.3, 13.4, 13.5**

- [ ] 9. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 10. Implement FastAPI application and endpoints
  - [ ] 10.1 Create FastAPI app with middleware
    - Setup FastAPI application with CORS
    - Add JWT authentication middleware
    - Add error handling middleware
    - Add logging middleware
    - _Requirements: 14.1_
  
  - [ ] 10.2 Implement POST /api/v1/upload endpoint
    - Accept multipart/form-data video upload
    - Validate file format and size
    - Check user quota
    - Upload to S3 with encryption
    - Return job ID and status URL
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6, 1.7_
  
  - [ ]* 10.3 Write unit tests for upload endpoint
    - Test valid uploads
    - Test invalid format rejection
    - Test size limit enforcement
    - Test quota enforcement
    - _Requirements: 1.1, 1.2, 1.3, 1.7_
  
  - [ ] 10.4 Implement POST /api/v1/segment endpoint
    - Accept segmentation point and mode
    - Validate video exists and user owns it
    - Create processing job in Redis
    - Start Step Functions execution
    - _Requirements: 2.2, 2.5_
  
  - [ ]* 10.5 Write unit tests for segment endpoint
    - Test valid segmentation requests
    - Test invalid video ID
    - Test unauthorized access
    - _Requirements: 2.2, 2.5, 14.4_
  
  - [ ] 10.6 Implement GET /api/v1/status/{jobId} endpoint
    - Query job status from Redis
    - Return current status and progress
    - Include download URL if complete
    - _Requirements: 6.5_
  
  - [ ] 10.7 Implement GET /api/v1/download/{jobId} endpoint
    - Verify job is complete
    - Verify user owns job
    - Generate CloudFront signed URL
    - Track download event
    - _Requirements: 8.1, 8.4, 14.4_
  
  - [ ]* 10.8 Write property test for download tracking
    - **Property 23: Download Event Tracking**
    - **Validates: Requirements 8.4**
  
  - [ ] 10.9 Implement DELETE /api/v1/video/{videoId} endpoint
    - Verify user owns video
    - Delete video from S3
    - Delete associated jobs
    - Schedule KMS key deletion
    - _Requirements: 12.2, 9.5_
  
  - [ ]* 10.10 Write property tests for data deletion
    - **Property 25: Encryption Key Deletion**
    - **Property 35: User Data Deletion**
    - **Validates: Requirements 9.5, 12.2**

- [ ] 11. Implement AWS Lambda handlers
  - [ ] 11.1 Create Lambda handler for FastAPI using Mangum
    - Wrap FastAPI app with Mangum adapter
    - Configure Lambda environment variables
    - _Requirements: All API requirements_
  
  - [ ] 11.2 Create validate-job Lambda function
    - Validate job parameters from Step Functions
    - Check video exists in S3
    - Verify segmentation point is within bounds
    - _Requirements: 6.1_
  
  - [ ] 11.3 Create extract-frames Lambda function
    - Extract video metadata using FFmpeg
    - Store metadata in Redis
    - _Requirements: 3.1_
  
  - [ ] 11.4 Create queue-job Lambda function
    - Add job to Redis queue
    - Update job status to queued
    - _Requirements: 6.2_
  
  - [ ] 11.5 Create check-status Lambda function
    - Query job status from Redis
    - Return status to Step Functions
    - _Requirements: 6.5_
  
  - [ ] 11.6 Create notify-user Lambda function
    - Send appropriate notification based on job status
    - _Requirements: 13.1, 13.2, 13.3_
  
  - [ ] 11.7 Create handle-failure Lambda function
    - Log error details to CloudWatch
    - Update job status to failed
    - Send failure notification
    - _Requirements: 6.4, 13.3_

- [ ] 12. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 13. Implement Step Functions state machine
  - [ ] 13.1 Create Step Functions definition using AWS CDK
    - Define all states (Validate, Extract, Queue, Wait, Check, Notify, HandleFailure)
    - Configure retry logic with exponential backoff
    - Configure error handling and transitions
    - _Requirements: 6.1, 6.2, 6.3, 6.4_
  
  - [ ]* 13.2 Write property tests for pipeline behavior
    - **Property 17: Pipeline State Transitions**
    - **Property 18: Retry Logic with Exponential Backoff**
    - **Property 19: Job Completion and Output Storage**
    - **Validates: Requirements 6.2, 6.3, 6.6**

- [ ] 14. Implement SAM3 segmentation engine
  - [ ] 14.1 Create SAM3 model loader and inference wrapper
    - Download SAM3 model weights
    - Create SAM3Segmenter class with segment and track methods
    - Implement frame extraction at 30fps using FFmpeg
    - _Requirements: 3.1, 3.2_
  
  - [ ] 14.2 Implement temporal mask propagation
    - Create track_object function using SAM3 temporal tracking
    - Generate binary masks for all frames
    - Handle tracking failures gracefully
    - _Requirements: 3.3, 3.4, 3.5_
  
  - [ ]* 14.3 Write property tests for SAM3 tracking
    - **Property 10: Frame Extraction Rate**
    - **Property 11: Binary Mask Generation**
    - **Property 12: Mask Sequence Completeness**
    - **Property 13: Tracking Failure Resilience**
    - **Validates: Requirements 3.1, 3.3, 3.4, 3.5, 3.6**
  
  - [ ] 14.4 Implement mask sequence output
    - Save masks as PNG files to S3
    - Create mask sequence metadata file
    - _Requirements: 3.6_

- [ ] 15. Implement Privacy Blur processing
  - [ ] 15.1 Create Gaussian blur processor
    - Implement apply_privacy_blur function using OpenCV
    - Use 99x99 kernel size for blur
    - Apply blur only to masked regions
    - _Requirements: 4.1, 4.2, 4.3_
  
  - [ ]* 15.2 Write property test for unmasked region preservation
    - **Property 14: Unmasked Region Preservation in Blur Mode**
    - **Validates: Requirements 4.3**
  
  - [ ] 15.3 Implement video encoding for blur mode
    - Encode processed frames to MP4 using FFmpeg
    - Preserve original resolution and frame rate
    - _Requirements: 4.4, 4.5_
  
  - [ ]* 15.4 Write property test for video property preservation
    - **Property 15: Video Property Preservation in Blur Mode**
    - **Validates: Requirements 4.5**

- [ ] 16. Implement ProPainter inpainting engine
  - [ ] 16.1 Create ProPainter model loader and inference wrapper
    - Download ProPainter model weights
    - Create ProPainterEngine class with inpaint method
    - _Requirements: 5.1_
  
  - [ ] 16.2 Implement temporal coherent inpainting
    - Apply inpainting to masked regions across all frames
    - Use temporal coherence for smooth transitions
    - _Requirements: 5.1, 5.2, 5.3_
  
  - [ ] 16.3 Implement video encoding for eraser mode
    - Encode inpainted frames to MP4 using FFmpeg
    - Preserve original resolution and frame rate
    - _Requirements: 5.4, 5.5_
  
  - [ ]* 16.4 Write property test for video property preservation
    - **Property 16: Video Property Preservation in Eraser Mode**
    - **Validates: Requirements 5.5**

- [ ] 17. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 18. Implement GPU worker process
  - [ ] 18.1 Create worker main loop
    - Implement Redis job polling with BLPOP
    - Implement idle timeout checking (5 minutes)
    - _Requirements: 7.3, 7.5_
  
  - [ ]* 18.2 Write property test for worker polling
    - **Property 20: Worker Job Polling**
    - **Validates: Requirements 7.3**
  
  - [ ] 18.3 Implement job processing pipeline in worker
    - Download video from S3
    - Run SAM3 tracking
    - Apply blur or inpainting based on mode
    - Encode output video
    - Upload to S3
    - Update job status in Redis
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 4.1, 5.1, 7.4_
  
  - [ ] 18.4 Implement spot interruption handling
    - Listen for EC2 spot interruption notices
    - Save progress and requeue job on interruption
    - _Requirements: 7.6_
  
  - [ ]* 18.5 Write property test for spot interruption
    - **Property 21: Spot Interruption Job Requeue**
    - **Validates: Requirements 7.6**
  
  - [ ] 18.6 Implement error handling in worker
    - Handle GPU out of memory errors
    - Handle SAM3 segmentation failures
    - Handle encoding failures
    - Log all errors to CloudWatch
    - _Requirements: Error Handling section_

- [ ] 19. Implement auto scaling logic
  - [ ] 19.1 Create auto scaling Lambda function
    - Monitor Redis queue depth
    - Launch GPU workers when depth > 10
    - Terminate idle workers when depth < 5
    - Maintain min 1, max 50 instances
    - _Requirements: 7.1, 10.2, 10.3, 10.4_
  
  - [ ]* 19.2 Write property tests for auto scaling
    - **Property 26: GPU Worker Scaling Up**
    - **Property 27: GPU Worker Scaling Down**
    - **Property 28: Worker Instance Count Boundaries**
    - **Validates: Requirements 10.2, 10.3, 10.4**

- [ ] 20. Implement monitoring and observability
  - [ ] 20.1 Create CloudWatch metrics emitter
    - Emit pipeline state execution times
    - Emit GPU utilization metrics
    - Emit API latency metrics
    - _Requirements: 15.1, 15.2, 15.4_
  
  - [ ]* 20.2 Write property tests for metrics
    - **Property 45: Pipeline State Execution Time Logging**
    - **Property 46: GPU Utilization Metrics**
    - **Property 48: High Latency Alarm**
    - **Validates: Requirements 15.1, 15.2, 15.4**
  
  - [ ] 20.3 Implement error logging with context
    - Log errors with stack traces to CloudWatch
    - Include job ID, user ID, and state in context
    - _Requirements: 15.3_
  
  - [ ]* 20.4 Write property test for error logging
    - **Property 47: Error Logging with Stack Traces**
    - **Validates: Requirements 15.3**
  
  - [ ] 20.5 Create CloudWatch alarms
    - High latency alarm (>1 second)
    - Spot interruption rate alarm (>20%)
    - Redis connection failure alarm
    - _Requirements: 15.4, 15.5_
  
  - [ ]* 20.6 Write property test for spot interruption alarm
    - **Property 49: Spot Interruption Rate Alarm**
    - **Validates: Requirements 15.5**

- [ ] 21. Implement GDPR compliance features
  - [ ] 21.1 Implement regional data storage
    - Detect user location from request
    - Route EU users to EU S3 buckets
    - _Requirements: 12.1_
  
  - [ ]* 21.2 Write property test for EU data residency
    - **Property 34: EU Data Residency**
    - **Validates: Requirements 12.1**
  
  - [ ] 21.3 Implement data access audit logging
    - Log all video data access events
    - Include user ID, resource ID, timestamp, accessor
    - _Requirements: 12.5_
  
  - [ ]* 21.4 Write property test for audit logging
    - **Property 36: Data Access Audit Logging**
    - **Validates: Requirements 12.5**

- [ ] 22. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 23. Implement React frontend
  - [ ] 23.1 Setup React project with TypeScript
    - Create React app with TypeScript template
    - Install dependencies (Plyr.js, Axios, TailwindCSS)
    - Configure build for S3 deployment
    - _Requirements: 2.1_
  
  - [ ] 23.2 Create video upload component
    - Implement file input with drag-and-drop
    - Show upload progress
    - Display validation errors
    - _Requirements: 1.1, 1.2, 1.3, 1.7_
  
  - [ ] 23.3 Create interactive video player component
    - Integrate Plyr.js video player
    - Implement click event handler for segmentation points
    - Display visual feedback for selected points
    - _Requirements: 2.1, 2.2, 2.3_
  
  - [ ]* 23.4 Write property test for segmentation point capture
    - **Property 7: Segmentation Point Capture**
    - **Validates: Requirements 2.2**
  
  - [ ] 23.5 Create processing mode selector
    - Display Privacy Blur and Magic Eraser options
    - Show mode descriptions and examples
    - _Requirements: 2.4_
  
  - [ ] 23.6 Create job status polling component
    - Poll status endpoint every 2 seconds
    - Display progress bar and current state
    - Show download button when complete
    - _Requirements: 6.5_
  
  - [ ] 23.7 Implement authentication flow
    - Create login/signup forms
    - Store JWT token in localStorage
    - Add token to all API requests
    - _Requirements: 14.1_
  
  - [ ] 23.8 Create user dashboard
    - Display user tier and quota
    - Show processing history
    - Display upgrade options for free tier
    - _Requirements: 11.1, 11.2, 11.4_

- [ ] 24. Implement AWS CDK infrastructure
  - [ ] 24.1 Create S3 bucket stacks
    - Define upload, output, and frontend buckets
    - Configure encryption with KMS
    - Configure lifecycle policies
    - Configure CORS for frontend access
    - _Requirements: 1.4, 8.5, 9.1, 9.2_
  
  - [ ] 24.2 Create CloudFront distribution stack
    - Configure origins for S3 buckets
    - Configure cache behaviors
    - Setup SSL certificate with ACM
    - Configure signed URL key pair
    - _Requirements: 8.1, 8.2, 8.3_
  
  - [ ] 24.3 Create ElastiCache Redis stack
    - Configure Redis cluster with Multi-AZ
    - Setup security groups
    - Configure backup retention
    - _Requirements: 6.5_
  
  - [ ] 24.4 Create Lambda function stacks
    - Deploy FastAPI Lambda with Mangum
    - Deploy Step Functions Lambda handlers
    - Deploy auto scaling Lambda
    - Configure environment variables and IAM roles
    - _Requirements: All Lambda requirements_
  
  - [ ] 24.5 Create Step Functions stack
    - Deploy state machine definition
    - Configure IAM roles for Lambda invocation
    - _Requirements: 6.1, 6.2, 6.3, 6.4_
  
  - [ ] 24.6 Create EC2 Auto Scaling Group stack
    - Configure launch template for GPU workers
    - Setup spot instance requests
    - Configure auto scaling policies
    - Setup user data script for worker initialization
    - _Requirements: 7.1, 7.2, 10.2, 10.3, 10.4_
  
  - [ ] 24.7 Create API Gateway stack
    - Configure REST API with Lambda integration
    - Setup custom domain
    - Configure throttling and quotas
    - _Requirements: All API requirements_
  
  - [ ] 24.8 Create CloudWatch stack
    - Create log groups for all services
    - Create metric namespaces
    - Create alarms
    - _Requirements: 15.1, 15.2, 15.3, 15.4, 15.5_

- [ ] 25. Integration and deployment
  - [ ] 25.1 Deploy infrastructure to AWS
    - Run CDK deploy for all stacks
    - Verify all resources are created
    - Test connectivity between services
  
  - [ ] 25.2 Deploy frontend to S3
    - Build React app for production
    - Upload to S3 frontend bucket
    - Invalidate CloudFront cache
  
  - [ ] 25.3 Deploy GPU worker AMI
    - Create custom AMI with SAM3 and ProPainter
    - Install CUDA, PyTorch, OpenCV, FFmpeg
    - Configure worker startup script
    - Update launch template with AMI ID
    - _Requirements: 7.2_

- [ ] 26. Final checkpoint - End-to-end testing
  - Run end-to-end integration tests
  - Verify all correctness properties hold
  - Test auto scaling behavior
  - Test spot interruption handling
  - Verify GDPR compliance features
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional property-based tests and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation throughout implementation
- Property tests validate universal correctness properties with 100+ iterations
- Unit tests validate specific examples and edge cases
- The implementation uses Python for backend, TypeScript for frontend, and AWS CDK for infrastructure
- GPU workers run on EC2 G5 Spot Instances for 70% cost savings
- All video data is encrypted with AES-256 using AWS KMS
- Auto-deletion after 24 hours ensures GDPR compliance
