# Form 21P-0516-1: Comprehensive Retry & Notification Analysis with GitHub URLs

## Executive Summary

This document provides a detailed analysis of the retry mechanisms and notification logic implemented for form 21P-0516-1 (Survivors Pension) in the VA.gov platform. All findings are validated through the GitHub codebase with direct links to source files.

## Key Findings

✅ **Retry Logic**: Implemented via Sidekiq background jobs with configurable retry settings  
✅ **Notification System**: Comprehensive email notification system via VA Notify  
✅ **Status Polling**: Automated background monitoring of form submission status  
⚠️ **Form 21P-0516-1 Status**: Supported for scanned uploads but missing email templates  

---

## 1. Retry Mechanisms

### 1.1 Notification Email Job Retries

**File**: [`app/sidekiq/simple_forms_api/notification/send_notification_email_job.rb`](https://github.com/department-of-veterans-affairs/vets-api/blob/master/app/sidekiq/simple_forms_api/notification/send_notification_email_job.rb)

```ruby
sidekiq_options retry: 10, backtrace: true
```

**Retry Configuration**:
- **Maximum Retries**: 10 attempts
- **Backoff Strategy**: Exponential backoff (Sidekiq default)
- **Error Tracking**: Full backtrace enabled
- **Job Type**: Email notification delivery

### 1.2 Benefits Intake Status Job

**File**: [`app/sidekiq/benefits_intake_status_job.rb`](https://github.com/department-of-veterans-affairs/vets-api/blob/master/app/sidekiq/benefits_intake_status_job.rb)

```ruby
sidekiq_options retry: false
```

**Configuration**:
- **Retries**: Disabled (retry: false)
- **Reasoning**: Status polling job - failure doesn't require retry as it runs periodically
- **Batch Size**: Configurable via settings (default: 1000)
- **SLA Monitoring**: 10-day threshold for stale submissions

---

## 2. Notification System Architecture

### 2.1 Primary Controller Integration

**File**: [`modules/simple_forms_api/app/controllers/simple_forms_api/v1/scanned_form_uploads_controller.rb`](https://github.com/department-of-veterans-affairs/vets-api/blob/master/modules/simple_forms_api/app/controllers/simple_forms_api/v1/scanned_form_uploads_controller.rb)

**Immediate Confirmation Email**:
```ruby
send_confirmation_email(params, confirmation_number) if status == 200
```

**Triggers**: 
- ✅ Immediate confirmation on successful upload
- ✅ Scheduled delivery at 9 AM Eastern Time

### 2.2 Email Service Implementation

**File**: [`modules/simple_forms_api/app/services/simple_forms_api/notification/email.rb`](https://github.com/department-of-veterans-affairs/vets-api/blob/master/modules/simple_forms_api/app/services/simple_forms_api/notification/email.rb)

**Key Features**:
- Template-based email system
- Feature flag control
- Personalization data injection
- MPI integration for user data

### 2.3 Form Upload Email Service

**File**: [`modules/simple_forms_api/app/services/simple_forms_api/notification/form_upload_email.rb`](https://github.com/department-of-veterans-affairs/vets-api/blob/master/modules/simple_forms_api/app/services/simple_forms_api/notification/form_upload_email.rb)

**Form 21P-0516-1 Support**:
```ruby
SUPPORTED_FORMS = %w[
  21P-4185  21-651    21-0304   21-8960   21P-4706c 21-4140
  21P-4718a 21-4193   21-0788   21-8951-2 21-674b   21-2680
  21-0779   21-4192   21-509    21-686c   21-8940
  21P-0516-1  # ✅ CONFIRMED SUPPORTED
  21P-0517-1  21P-0518-1  21P-0519C-1  21P-0519S-1
  21P-530a  21P-8049
].freeze
```

**Email Templates**:
```ruby
TEMPLATE_IDS = {
  confirmation: template_root.form_upload_confirmation_email,
  error: template_root.form_upload_error_email,
  received: template_root.form_upload_received_email
}
```

### 2.4 Template Configuration

**File**: [`modules/simple_forms_api/app/services/simple_forms_api/notification/template_ids.yml`](https://github.com/department-of-veterans-affairs/vets-api/blob/master/modules/simple_forms_api/app/services/simple_forms_api/notification/template_ids.yml)

**⚠️ Critical Finding**: Form 21P-0516-1 is **NOT** in the template_ids.yml file, meaning no email templates are configured yet.

**Existing Templates** (excerpt):
```yaml
vba_21p_0847:
  confirmation: form21p_0847_confirmation_email
  error: form21p_0847_error_email
  received: form21p_0847_received_email
# 21P-0516-1 templates are MISSING
```

---

## 3. Status Polling & State Management

### 3.1 Form Submission Attempt Model

**File**: [`app/models/form_submission_attempt.rb`](https://github.com/department-of-veterans-affairs/vets-api/blob/master/app/models/form_submission_attempt.rb)

**State Machine**:
```ruby
aasm do
  state :pending, initial: true
  state :failure, :success, :vbms
  state :manually
  
  event :fail do
    after do
      simple_forms_enqueue_result_email if should_send_simple_forms_email
    end
    transitions from: :pending, to: :failure
  end
  
  event :vbms do
    after do
      simple_forms_enqueue_result_email if should_send_simple_forms_email
    end
    transitions from: :pending, to: :vbms
    transitions from: :success, to: :vbms
  end
end
```

**Notification Triggers**:
- ✅ **Failure State**: Triggers error notification email
- ✅ **VBMS Success**: Triggers success notification email
- ✅ **Feature Flag Controlled**: `simple_forms_email_notifications`

### 3.2 Background Status Polling

**File**: [`app/sidekiq/benefits_intake_status_job.rb`](https://github.com/department-of-veterans-affairs/vets-api/blob/master/app/sidekiq/benefits_intake_status_job.rb)

**Status Monitoring Flow**:
```ruby
def perform
  pending_form_submission_attempts = FormSubmissionAttempt.where(aasm_state: 'pending')
  
  pending_form_submission_attempts.each_slice(batch_size) do |batch|
    batch_uuids = batch.map(&:benefits_intake_uuid)
    response = intake_service.bulk_status(uuids: batch_uuids)
    handle_response(response)
  end
end
```

**Status Handling**:
- **`vbms`**: Successfully processed into VBMS → trigger success email
- **`error`**: Processing error → trigger error email  
- **`expired`**: Upload window expired (15 min) → trigger error email
- **`pending`**: Still processing → continue monitoring

---

## 4. Benefits Intake Integration

### 4.1 Core Service

**File**: [`lib/lighthouse/benefits_intake/service.rb`](https://github.com/department-of-veterans-affairs/vets-api/blob/master/lib/lighthouse/benefits_intake/service.rb)

**API Integration**:
- Upload request handling
- Status polling via bulk_status
- Metadata validation
- Error handling and reporting

---

## 5. Feature Flags & Configuration

### 5.1 Email Notification Feature Flag

**Validation Function** (from email.rb):
```ruby
def flipper?
  number = form_number
  number = 'vba_21_0966' if form_number.start_with? 'vba_21_0966'
  Flipper.enabled?(:"form#{number.gsub('vba_', '')}_confirmation_email")
end
```

**Expected Flag for 21P-0516-1**: `form21p_0516_1_confirmation_email`

### 5.2 Simple Forms Email Notifications

**Flag Check** (from form_submission_attempt.rb):
```ruby
def should_send_simple_forms_email
  simple_forms_form_number && Flipper.enabled?(:simple_forms_email_notifications)
end
```

**Required Flags**:
- ✅ `simple_forms_email_notifications` (global simple forms email toggle)
- ⚠️ `form21p_0516_1_confirmation_email` (form-specific email toggle)

---

## 6. Monitoring & Alerting

### 6.1 Datadog Integration

**Metrics Collection**:
```ruby
StatsD.increment("api.benefits_intake.submission_status.#{form_id}.#{result}")
StatsD.increment("api.benefits_intake.submission_status.all_forms.#{result}")
```

**Alert Types**:
- `success`: Successful VBMS integration
- `failure`: Processing failures
- `stale`: Submissions exceeding SLA (10 days)
- `silent_failure`: Email notification failures

### 6.2 Structured Logging

**Error Logging**:
```ruby
Rails.logger.error('BenefitsIntakeStatusJob', 
  result:, form_id:, uuid:, time_to_transition:, error_message:)
```

**Status Change Logging**:
```ruby
Rails.logger.info('Form Submission Attempt State change',
  from_state:, to_state:, event:, form_type:, benefits_intake_uuid:)
```

---

## 7. Current Status for Form 21P-0516-1

### 7.1 What's Working ✅

1. **Scanned Form Upload**: Full support via scanned_form_uploads_controller
2. **Status Polling**: Automated monitoring via BenefitsIntakeStatusJob
3. **State Management**: AASM state machine with proper transitions
4. **Form Support**: Listed in FormUploadEmail SUPPORTED_FORMS
5. **Retry Logic**: 10-retry configuration for email delivery
6. **Monitoring**: Datadog metrics and structured logging

### 7.2 What's Missing ⚠️

1. **Email Templates**: No templates in template_ids.yml
2. **Feature Flags**: Form-specific feature flags may not be configured
3. **Template Content**: Actual email template content in VA Notify

### 7.3 What Needs Investigation ❓

1. **Digital Form Support**: Whether the form supports digital submission
2. **Template Configuration**: VA Notify template setup
3. **Feature Flag Status**: Current flag configuration in production

---

## 8. Implementation Gaps & Next Steps

### 8.1 Required for Full Notification Support

1. **Add Email Templates**:
   ```yaml
   # Add to template_ids.yml
   vba_21p_0516_1:
     confirmation: form21p_0516_1_confirmation_email
     error: form21p_0516_1_error_email
     received: form21p_0516_1_received_email
   ```

2. **Configure Feature Flags**:
   - Enable `simple_forms_email_notifications`
   - Enable `form21p_0516_1_confirmation_email`

3. **Create VA Notify Templates**:
   - Design confirmation email template
   - Design error notification template
   - Design received notification template

### 8.2 Recommended Monitoring Enhancements

1. **Custom Dashboards**: Form-specific monitoring in Datadog
2. **Alert Configuration**: Set up alerts for 21P-0516-1 failures
3. **SLA Tracking**: Monitor form-specific processing times

---

## 9. Key GitHub Files Reference

| Component | File Path | GitHub URL |
|-----------|-----------|------------|
| **Main Controller** | `modules/simple_forms_api/app/controllers/simple_forms_api/v1/scanned_form_uploads_controller.rb` | [View](https://github.com/department-of-veterans-affairs/vets-api/blob/master/modules/simple_forms_api/app/controllers/simple_forms_api/v1/scanned_form_uploads_controller.rb) |
| **Email Service** | `modules/simple_forms_api/app/services/simple_forms_api/notification/email.rb` | [View](https://github.com/department-of-veterans-affairs/vets-api/blob/master/modules/simple_forms_api/app/services/simple_forms_api/notification/email.rb) |
| **Form Upload Email** | `modules/simple_forms_api/app/services/simple_forms_api/notification/form_upload_email.rb` | [View](https://github.com/department-of-veterans-affairs/vets-api/blob/master/modules/simple_forms_api/app/services/simple_forms_api/notification/form_upload_email.rb) |
| **Template Config** | `modules/simple_forms_api/app/services/simple_forms_api/notification/template_ids.yml` | [View](https://github.com/department-of-veterans-affairs/vets-api/blob/master/modules/simple_forms_api/app/services/simple_forms_api/notification/template_ids.yml) |
| **Email Job** | `app/sidekiq/simple_forms_api/notification/send_notification_email_job.rb` | [View](https://github.com/department-of-veterans-affairs/vets-api/blob/master/app/sidekiq/simple_forms_api/notification/send_notification_email_job.rb) |
| **Status Job** | `app/sidekiq/benefits_intake_status_job.rb` | [View](https://github.com/department-of-veterans-affairs/vets-api/blob/master/app/sidekiq/benefits_intake_status_job.rb) |
| **Form Model** | `app/models/form_submission_attempt.rb` | [View](https://github.com/department-of-veterans-affairs/vets-api/blob/master/app/models/form_submission_attempt.rb) |
| **Benefits Intake** | `lib/lighthouse/benefits_intake/service.rb` | [View](https://github.com/department-of-veterans-affairs/vets-api/blob/master/lib/lighthouse/benefits_intake/service.rb) |

---

## 10. Summary

Form 21P-0516-1 has **comprehensive retry and notification infrastructure** in place but requires **email template configuration** to enable full notification functionality. The system includes:

- ✅ **10-retry email delivery system**
- ✅ **Automated status polling**  
- ✅ **State-based notification triggers**
- ✅ **Comprehensive monitoring & alerting**
- ⚠️ **Missing email templates for 21P-0516-1**

Once email templates are configured, users will receive notifications when their form reaches VBMS or encounters processing errors.
