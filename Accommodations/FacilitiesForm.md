# Facilities Form
***Micro-service | Fg-Admin-Panel-Frontend | Accommodations Module***

# Component: FacilitiesForm

## 1. Component Name

**FacilitiesForm.tsx** — A dynamic form component for updating either facilities or permitted passenger types for a specific accommodation on a vessel in the admin panel.

---

## 2. Overall Description

This component allows administrators to assign or update the **facilities** and **passenger types permitted** for a selected accommodation. It is used inside the `Accommodation` module of the admin panel on a ferry ticketing website.

The component is driven by the `type` prop, which determines whether the form updates facilities or passenger types. It leverages `react-hook-form` for form control, makes API calls to update the backend, and uses checkboxes for selection.

---

## 3. Breakdown of the Component

## Chunk 1: Import Statements

```tsx
import {
  Button, Checkbox, CircularProgress, FormControlLabel, Grid
} from '@mui/material'
import React, { useEffect, useMemo, useState } from 'react'
import { Controller, useForm } from 'react-hook-form'
import toast from 'react-hot-toast'
import { patchAccommodationsWithoutId } from 'src/api/accommodations.service'
import { AccommodationsT } from 'src/types/accommodations.types'
import { AC_URL } from '../../utils/common.constants'
```

**Explanation:**
Standard imports for Material UI components, hooks, API services, types, and constants used in the form. `react-hook-form` handles form state, and toast provides feedback messages.

## Chunk 2: Component Props and State
```tsx
const FacilitiesForm = ({
  data,
  id,
  accommodationData,
  type,
  fetchFunc
}: {
  data: any[]
  id: number
  accommodationData: AccommodationsT | {}
  fetchFunc: () => void
  type: 'FACILIITIES' | 'PASSENGER_TYPES_PERMITTED'
}) => {
  const [submitting, setSubmitting] = useState(false)
  const URL = type === 'FACILIITIES' ? AC_URL.FACILITIES_AVAILABLE : AC_URL.PASSENGER_TYPES_PERMITTED
  const { control, handleSubmit, reset } = useForm()
```

**Explanation:**
Defines props including:
- `data`: the full list of facilities or passenger types.
- `id`: ID of the accommodation being edited.
- `accommodationData`: current data tied to the accommodation.
- `fetchFunc`: a callback to refresh data after submission.
- `type`: determines whether the form edits facilities or passenger types.

Initializes form handling and a loading state.

## Chunk 3: Determine Mapped Values
```tsx
  let dataMapping
  let idMapping

  switch (type) {
    case 'FACILIITIES':
      dataMapping = (accommodationData as any)?.accommodation_facility_mappings ?? []
      idMapping = dataMapping?.map((item: any) => item.facility_id) ?? []
      break
    case 'PASSENGER_TYPES_PERMITTED':
      dataMapping = (accommodationData as any)?.accommodation_passenger_type_mappings ?? []
      idMapping = dataMapping?.map((item: any) => item.passenger_type_id) ?? []
      break
  }
  ```

**Explanation:**
This block conditionally extracts existing mappings from the provided accommodation data based on the `type`, generating a list of already-selected item IDs for pre-filling the checkboxes.

## Chunk 4: Set Default Checkbox Values
```tsx
  const defaultValues = useMemo(() => {
    return data.reduce((acc, item) => {
      acc[item.id] = idMapping.includes(item.id)
      return acc
    }, {})
  }, [data, idMapping])
```
**Explanation:**
Generates a `defaultValues` object indicating which checkboxes should be pre-selected by checking if their ID exists in the current mappings.

## Chunk 5: Reset Form on Mount
```tsx
  useEffect(() => {
    reset(defaultValues)
  }, [])
```

**Explanation:**
When the component mounts, the form is reset using the precomputed default values. This ensures the checkboxes reflect the existing state of the accommodation.

## Chunk 6: Form Submission Handler
```tsx
  const onSubmit = async (submitData: any) => {
    setSubmitting(true)
    const arr: number[] = []
    const updatedFacility = data
      .filter(data => String(data.id) in submitData && submitData[String(data.id)] === true)
      .map(data => arr.push(data.id))

    let payload

    switch (type) {
      case 'FACILIITIES':
        payload = {
          accommodation_id: id,
          facility_ids: arr
        }
        break
      case 'PASSENGER_TYPES_PERMITTED':
        payload = {
          passenger_types_ids: arr,
          accommodation_id: id
        }
    }

    try {
      const response = await patchAccommodationsWithoutId(URL, payload)

      if (response.status === HttpStatusCode.Created || response.status === HttpStatusCode.Ok) {
        toast.success('Record was updated')
        if (fetchFunc) fetchFunc()
      } else {
        toast.error('Something went wrong, please try again')
      }
    } catch (e: any) {
      if (e.response && e.response.status === 409) {
        toast.error('One or more fields have matching or duplicate values.')
      } else {
        toast.error(`Something went wrong ${e.message || ''}`)
      }
    } finally {
      setSubmitting(false)
    }
  }

```

**Explanation:**
- Collects selected checkbox IDs.
- Builds the payload differently depending on the type.
- Makes an API call to update the backend.
- Displays success or error messages accordingly.

## Chunk 7: Form Rendering
```tsx
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Grid container spacing={2}>
        {data.map((item: any) => (
          <Grid item xs={12} sm={5} key={item.id}>
            <Controller
              key={item.id}
              name={String(item.id)}
              control={control}
              defaultValue={defaultValues[item.id] || false}
              render={({ field }) => (
                <FormControlLabel
                  control={
                    <Checkbox {...field} checked={field.value} onChange={e => field.onChange(e.target.checked)} />
                  }
                  label={item.name}
                />
              )}
            />
          </Grid>
        ))}
        <Grid item xs={12}>
          <Button type='submit' variant='contained' disabled={submitting}>
            {!submitting ? 'Save' : <CircularProgress size={20} />}
          </Button>
        </Grid>
      </Grid>
    </form>
  )
}
```

**Explanation:**
Renders a form with dynamically generated checkboxes for each item in `data`. Uses `react-hook-form`’s `Controller` to link checkboxes with form state. The submit button shows a loader while submitting.

---

## 4. Return Value / Possible Responses

This component does not return a value explicitly. However, it:
- Submits selected IDs to the API.
- Calls the `fetchFunc` callback after successful submission to refresh parent data.
- Displays success/error toasts based on API response.

## 5. Error Handling
- **409 Conflict:** Displays an error toast indicating duplicate or matching fields.
- **Any Other Error:** Catches unexpected errors and displays a toast with the error message (if available).
- **Submission Disabled:** During submission, the form prevents double submissions by disabling the button and showing a loading spinner.

---

## Key Features

- Dynamically handles both "Facilities" and "Passenger Types Permitted" based on the `type` prop.
- Pre-selects checkboxes based on current accommodation data.
- Uses `react-hook-form` for minimal and efficient form state management.
- Optimistically updates the form using `useMemo` and `useEffect`.
- Submits data via `patchAccommodationsWithoutId` API service.
- Displays submission loading state using MUI `CircularProgress`.
- Uses toast notifications for success and error feedback.