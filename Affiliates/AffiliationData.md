# Affiliation data
***Micro-service | Fg-Admin-Panel-Frontend | Affiliates Module***

## Component Name
**AffiliationData.tsx** — — A form component for creating or updating affiliate profiles within the Affiliate Module of the ferry ticket booking system’s admin panel.

## Overall Description
This component renders the form used to add new affiliates or edit existing ones. It determines its mode (add vs. edit) based on the presence of `affiliate_id` from context. The form handles input fields for username, password, affiliate start date, commission type (fixed percentage vs. booking-range based), and dynamic booking range tables. Validation is enforced via a Yup schema and React Hook Form.

## Breakdown of the Component

## Chunk 1: Import Statements
```typescript
import { Grid } from '@mui/material'
import React, { useEffect, useState } from 'react'
import { Controller, useForm } from 'react-hook-form'
import CustomTextField from 'src/@core/components/mui/text-field'
import CustomLabel from 'src/components/common/base/CustomLabel'
import BookingRangeTable from './BookingRangeTable'
import { useRouter } from 'next/router'
import AFFILIATES_URLS from 'src/routes/affiliates.routes'
import SaveAndCancelButton from 'src/components/common/base/Buttons/SaveAndCancelButton'
import RadioButtonsWithController from 'src/components/common/customInput/RadioButtonsWithController'
import TextFieldWithController from 'src/components/common/customInput/TextFieldWithController'
import DataInputField from 'src/components/common/customInput/DataInputField'
import { upsertGeneralAffliatesData } from 'src/api/affiliates.service'
import { HttpStatusCode } from 'axios'
import toast from 'react-hot-toast'
import * as title from 'src/utils/common.constants'
import { useAffiliateContext } from 'src/context/AffiliatesContext'
import { yupResolver } from '@hookform/resolvers/yup'
import validation_schema from 'src/validation-schemas/affiliations.schema'
```

**Explanation:**
- **React & Hooks:** Manage component state (useState) and lifecycle - (useEffect).
- **React Hook Form:** Provides form control, validation integration via Yup.
- **MUI Grid:** Layout for responsive grid.
- **Custom Components:** Text fields, labels, buttons, and booking ranges table.
- **Router & URLs:** Next.js useRouter for navigation and route constants.
API & Toasts: Service call for upsert and toast notifications on success/error.
- **Context & Validation:** useAffiliateContext provides affiliate data and yupResolver enforces schema rules.

## Chunk 2: Form Setup & State

```typescript
const { control, handleSubmit, watch, reset, formState: { errors } } = useForm({
  resolver: yupResolver(validation_schema['AFFILIATION_DATA'])
})
const [showPassword, setShowPassword] = useState(false)
const [disabled, setDisabled] = useState(false)
const router = useRouter()
const {
  affiliate_id,
  affliateData,
  refetchAffiliates,
  setAffiliateId
} = useAffiliateContext()
```

**Explanation:**
- **useForm:** Initializes form methods with Yup validation schema.
- **showPassword & disabled:** Local UI states for toggling password visibility and disabling the form during submission.
- **Router & Context:** Access affiliate-related data and navigation.

## Chunk 3: Reset Form Effect

```typescript
const resetForm = () => {
  reset({
    commission_type: affliateData.commission_type,
    fixed_commission_percentage: affliateData.fixed_commission_percentage,
    inputs: affliateData?.commission_booking_ranges?.map(range => ({
      from_bookings: range.from_bookings || null,
      to_bookings: range.to_bookings || null,
      commission_percentage: range.commission_percentage || null
    }))
  })
}
useEffect(() => {
  resetForm()
}, [affliateData])
```

**Explanation:**
- **resetForm:** Populates form fields from existing affiliate data, including dynamic booking ranges.
- **useEffect:** Triggers resetForm whenever affliateData changes (e.g., when editing a different affiliate).

## Chunk 4: Handling Submission (onSubmit)

```typescript
const onSubmit = async (data) => {
  const payload = {
    commission_type: data.commission_type,
    fixed_commission_percentage: data.commission_type === 1 ? data.fixed_commission_percentage : null,
    commission_booking_ranges: /* filter and map logic */,  
    id: affiliate_id
  }
  if (!affiliate_id) delete payload.id

  setDisabled(true)
  try {
    const response = await upsertGeneralAffliatesData(AFFILIATES_URLS.AFFILIATION_DATA, payload)
    if (response.status === HttpStatusCode.Ok || response.status === HttpStatusCode.Created) {
      toast.success(affiliate_id ? title.SUCCESS_TITLE_AMENDED : 'Affiliate created')
      if (!affiliate_id) setAffiliateId(response.data.data.id)
      refetchAffiliates?.()
    } else {
      toast.error(title.ERROR_TITLE_AMENDED)
    }
  } catch (e) {
    toast.error(title.ERROR_TITLE_AMENDED + e.message)
  } finally {
    setDisabled(false)
  }
}
```

**Explanation:**
- **Payload Construction:** Conditionally includes fixed_commission_percentage or booking ranges.
- **Edit vs. Create:** Deletes id when creating a new affiliate.
- **Service Call & Notifications:** Calls API, shows toast messages, updates context, and refetches the affiliate list.

# Chunk 5: Form Rendering

```typescript
<form onSubmit={handleSubmit(onSubmit)}>
  <Grid container direction="column" gap={4}>
    {/* Username & Password Fields (disabled) */}
    <Controller name="username" /*...*/ />
    <Controller name="password" /*...*/ />
    {affiliate_id && (
      <Controller name="accepted_on" /* read‐only date */ />
    )}

    {/* Commission Type Radio Buttons */}
    <RadioButtonsWithController control={control} name="commission_type" /* options */ />

    {/* Conditional Commission Inputs */}
    {watch('commission_type') === 1 ? (
      <TextFieldWithController name="fixed_commission_percentage" /*...*/ />
    ) : (
      <BookingRangeTable control={control} errors={errors} />
    )}

    {/* Save / Create & Cancel Buttons */}
    <SaveAndCancelButton title2={affiliate_id ? 'Save' : 'Create'} disabled={disabled} onCancel={() => router.push(AFFILIATES_URLS.HOME)} />
  </Grid>
</form>
```

**Explanation:**
- Controlled inputs via Controller from React Hook Form.
- Disabled username/password fields for display only.
- Dynamic sections based on commission_type.
- Action buttons to save or cancel.

### Return Value
The component returns JSX that renders a fully controlled form. It includes loading of existing data, real‐time validation feedback, and dynamic form sections.

### Error Handling
- Validation Errors: Managed by React Hook Form and Yup; errors are passed into fields via errors.
- API Errors: Caught in onSubmit; displays toast messages combining a generic error title with the error message.

---

## Key Features
- Dual Mode (Add/Edit):
Determines if the form is in "Add" or "Edit" mode based on the presence of `affiliate_id`.

- Form Validation with Yup:
Uses `react-hook-form` with `yupResolver` to enforce a validation schema on form inputs.

- Dynamic Commission Type Handling:
  Supports both:
  - Fixed commission percentage
  - Commission based on booking ranges -- rendered conditionally based on the user's selection.

- Data Pre-fill for Edit Mode:
Automatically pre-fills form fields using affliateData when in edit mode.

- Disabled Inputs for Immutable Fields:
Username, password, and acceptance date fields are shown but non-editable to preserve integrity.

- Centralized API Call for Add/Edit:
Uses a single API (upsertGeneralAffliatesData) to handle both creation and update operations.

- User Feedback via Toast Notifications:
Success and error messages are shown using react-hot-toast for better user experience.

- Context Integration:
Leverages useAffiliateContext() to access and update shared affiliate-related state.

- Navigation Handling:
Uses Next.js useRouter to redirect users after saving or cancelling.

- Reusability & Modularity:
Employs multiple reusable custom components like:
 - TextFieldWithController
 - RadioButtonsWithController
 - BookingRangeTable
 - SaveAndCancelButton
