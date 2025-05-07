# Basic Details Section
***Micro-service |  Fg-Admin-Panel-Frontend | New Reservations Module*** 

## 1. Component Name
**BasicDetailsSection.tsx** — A section component that displays and manages basic route details for each booking leg within the Reservations detail page in the admin panel.

## 2 Overall Description
The `BasicDetailsSection` component is part of the Reservations module’s detail page in the ferry ticket website admin panel. When an administrator clicks the "eye" icon on the main Reservations grid, they land on a detail page where they can view and edit information related to individual booking legs (routes). This component renders each leg’s basic details inside an expandable accordion. It supports view and edit modes, allowing inline updates of route, date/time, company, vessel, status, ticket amount, and ticket status. It also provides buttons to add passengers, vehicles, and pets for each route, with conditional disabling based on availability.

---

## Breakdown of the Component

### Chunk 1: Import Statements

```tsx
import React, { useEffect, useMemo, useState } from 'react'
import { StyledBox } from '../../../components/StyledComponents'
import { Box, Grid, Tooltip, Typography } from '@mui/material'
import { getRouteData } from '../../columns'
import { LegT } from 'src/types/reservations.types'
import AccordionSection from '../../../components/AccordianSection'
import {
  EditableButtonGroup,
  EditableField,
  PassengerIcon,
  PetIcon,
  PlusIcon,
  VehicleIcon
} from 'src/components/new-reservations/general/methods/reservations.method'
import { Controller, useForm } from 'react-hook-form'
import useEssentialData from 'src/hooks/useEssentialData'
import { updateBookingLegStatus, updateReservationRouteBasicDetails } from 'src/api/reservations.service'
import StatusColumn from '../../columns/common/StatusColumn'
import {
  handleUpdateReservation,
  mapBasicDetailsData
} from 'src/components/new-reservations/general/utils/reservations.helper'
import { TABLES } from 'src/constants/common.constants'
import { useReservationContext } from 'src/context/ReservationContext'
import CustomButton from 'src/components/common/base/Buttons/CustomButton'
```

**Explanation:**
- React hooks (`useState`, `useEffect`, `useMemo`) manage local state, side effects, and memoization.
- MUI components (`Box`, `Grid`, `Tooltip`, `Typography`) build the layout and styles.
- `getRouteData` shapes raw route data into form fields.
- `AccordionSection`, `EditableField`, and `EditableButtonGroup` compose the accordion UI and inline editing controls.
- `react-hook-form` handles form state, validation, and submission.
- `useEssentialData` fetches dynamic dropdown options (e.g., vessels) based on the selected company.
- API functions (`updateBookingLegStatus`, `updateReservationRouteBasicDetails`) persist updates to the backend.
- `useReservationContext` provides availability info for pets and vehicles.

### Chunk 2: Component Signature & Props

```tsx
export type FormValues = { id: string; [key: string]: any }

const BasicDetailsSection = ({
  bookingId,
  route,
  isEdit,
  companyCode,
  setCompanyCode,
  fetchAllRoutes,
  routes,
  companyData,
  noPassengersExists,
  setTargetPetRouteId,
  setTargetPassengerRouteId,
  setTargetVehicleRouteId
}: {
  bookingId: string
  route: LegT
  isEdit: boolean
  companyCode: string
  setCompanyCode: (newCode: string) => void
  fetchAllRoutes: () => void
  routes: any[]
  companyData: EssentialDataProps[]
  noPassengersExists: boolean
  setTargetPetRouteId: (id: number) => void
  setTargetPassengerRouteId: (id: number) => void
  setTargetVehicleRouteId: (id: number) => void
}) => { ... }
```

**Explanation:**

- `bookingId`: Parent booking identifier.
- `route`: Data for a single leg (port codes, dates, vessel, amounts, status).
- `isEdit`: Toggles view vs edit mode.
- `companyCode` & `setCompanyCode`: Track selected ferry company to fetch vessel options.
- `fetchAllRoutes`: Reloads route list when entering edit mode.
- `routes`, `companyData`: Option lists for dropdowns.
- `noPassengersExists`: Disables pet button if no passengers.
- `setTarget*RouteId`: Callbacks to open add-passenger/vehicle/pet dialogs for this leg.

### Chunk 3: Data Preparation & Form Setup

```tsx
const [routeData, setRouteData] = useState(route)
const currentRoutesData = useMemo(
  () => getRouteData(bookingId, routeData, companyData, vesselData, routes),
  [bookingId, routeData, companyData, vesselData, routes]
)

const defaultValues = useMemo(() => {
  return currentRoutesData.reduce((acc, item) => {
    // initialize form fields based on field type
  }, {})
}, [routeData, currentRoutesData])

const { control, handleSubmit, watch, reset, formState: { isDirty, errors } } = useForm<FormValues>({
  defaultValues: { ferry_booking_detail_id: String(routeData.ferry_booking_detail_id), ...defaultValues }
})
```

**Explanation:**
- `routeData` holds the live data for the leg; resets when isEdit ends.
- `getRouteData` builds an array of field descriptors (labels, values, dropdown configs).
- `defaultValues` maps these descriptors to the initial form state.
- `useForm` initializes form control, dirtiness detection, and error tracking.

### Chunk 4: Side Effects

```tsx
useEffect(() => {
  if (!isEdit) setRouteData(route)
}, [route, isEdit])

useEffect(() => {
  if (isEdit) fetchAllRoutes()
}, [isEdit])

useEffect(() => {
  const selectedCompanyCode = watch('company')
  setCompanyCode(selectedCompanyCode)
}, [watch('company')])
```

**Explanation:**
- Resets `routeData` when exiting edit mode.
- Fetches fresh vessel and company data when entering edit mode.
- Syncs selected company dropdown to parent state for dependent data loads.

### Chunk 5: Submission Handler

```tsx
const onSubmit = async (data) => {
  setLoading(true)
  const payload = mapBasicDetailsData(data, routes, companyData, vesselData, route)
  const updatedLeg = await handleUpdateReservation(
    updateReservationRouteBasicDetails,
    bookingId,
    payload,
    'Basic details'
  )
  setRouteData(updatedLeg)
  reset({ ferry_booking_detail_id: String(updatedLeg.ferry_booking_detail_id), ...defaultValues })
  setLoading(false)
}
```

**Explanation:**
- Constructs payload from form values and helper utilities.
- Calls API to persist updates.
- Updates `routeData` and resets form to the new saved state.

### Chunk 6: Rendering & Editable Controls

```tsx
<StyledBox>
  <AccordionSection title={<Typography>Basic Details</Typography>}>
    <form onSubmit={handleSubmit(onSubmit)}>
      <Grid container spacing={2}>
        {currentRoutesData.map(item => (
          <Grid item xs={12} sm={6} key={item.id}>
            <Controller name={item.field} control={control} render={({ field }) => (
              <EditableField {...field} label={item.title} isEdit={isEdit} ... />
            )} />
          </Grid>
        ))}
      </Grid>
      <Box sx={{ display: 'flex', justifyContent: 'space-between' }}>
        <EditableButtonGroup isEdit={isEdit} />
        {!isEdit && <Add Buttons for Passenger, Vehicle, Pet with Tooltips />}  
      </Box>
    </form>
  </AccordionSection>
</StyledBox>
```

**Explanation:**
- Wraps fields in an accordion titled “Basic Details.”
- Maps each descriptor to an `EditableField`, which conditionally renders a dropdown, date/time inputs, or text fields.
- Displays Save/Cancel buttons in edit mode via `EditableButtonGroup`.
- Shows "Add Passenger/Vehicle/Pet" buttons in view mode, disabled based on availability with tooltips.

---

## 4. Return Value
The component returns JSX comprising a styled box containing an accordion. Inside, it renders a controlled form of basic route details and action buttons. It leverages React.memo for performance optimization to avoid unnecessary re-renders.

## 5. Error Handling
- Submission failures are caught in the `onSubmit` try/catch. On error, the form resets to its last known valid state.
- Validation errors for each field populate the `errors` object from `react-hook-form` and are passed into `EditableField` to show error highlighting.
- Disabled Add buttons show contextual tooltips when passenger, vehicle, or pet additions aren’t allowed.

---

## Key Features
- **Accordion UI for Leg Details:** Cleanly groups basic route information using an expandable section for each booking leg.
- **View & Edit Modes:** Seamless toggle between read-only view and inline editing for key booking leg fields.
- **Dynamic Dropdowns:** Company, vessel, and route fields auto-update based on selected values and live data using `useEssentialData`.
- **Add Entities Conditionally:** Buttons for adding passengers, vehicles, and pets appear contextually and respect current booking constraints.
- **Form State Management:** Robust handling of form control, validation, dirtiness tracking, and reset using `react-hook-form`.
- **Optimized Re-rendering:** Uses `useMemo`, `useEffect`, and memoized field descriptors to enhance performance.
- **Error Handling & Feedback:** Gracefully manages API failures and validation errors with clear visual cues and resets.

