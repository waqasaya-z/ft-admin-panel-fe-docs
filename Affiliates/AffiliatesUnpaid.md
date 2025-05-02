# Affiliates Unpaid (Unpaid per affiliate)
***Micro-service | Fg-Admin-Panel-Frontend | Affiliates Module***

## Component Name
**AffiliatesUnpaid.tsx** — A React component under the `Unpaid per affiliate` tab of the Affiliate module. It renders a data grid of affiliates with unpaid earnings and provides a clearance workflow (lock, complete, proceed to payment, pay checked).

## Overall Description
The `AffiliatesUnpaid` component manages and displays the clearance procedure for unpaid affiliate earnings. It allows an admin to:
- Set and lock cutoff date and minimum amount filters
- Fetch and list affiliates meeting criteria
- Review and update clearance status per affiliate
- Execute batch operations: lock clearance, complete clearance, proceed to payment, pay checked

It leverages MUI's Grid and DataGrid, custom hooks for API interactions, and a set of filters and actions in the toolbar.

---

## Breakdown of the Component
Below, each major section of the component is explained in detail.

## Chunk 1: Import Statements

```typescript
import { Grid } from '@mui/material'
import { GridSortModel } from '@mui/x-data-grid'
import React, { useEffect, useState } from 'react'
import MUIDataGrid from 'src/@core/components/mui/datagrid'
import { fetchAllGeneralAffliatesData } from 'src/api/affiliates.service'
import DatagridScrollWrapper from 'src/components/common/feature/DatagridScrollWrapper'
import AFFILIATES_URLS from 'src/routes/affiliates.routes'
import { useFetchData } from 'src/utils/helper'
import UnpaidAffiliatesColumns from './columns'
import AffiliatesUnpaidFilters, { MODE } from './AffiliatesUnpaidFilters'
import toast from 'react-hot-toast'
import useClearanceStatus from '../general/hooks/useClearanceStatus'
import useAppendClearanceData from '../general/hooks/useAppendClearanceData'
import useAppendAffiliateIds from '../general/hooks/useAppendAffiliateIds'
import { AffiliatesSortModel } from '../general/common/affliates.methods'
```

**Explanation:**
- **React imports** (useState, useEffect) for state and lifecycle.
- **MUI components** (Grid, DataGrid) for layout and tabular data.
- **Custom hooks:**
    - `useClearanceStatus` fetches current clearance record.
    - `useAppendClearanceData` updates/locks/ completes a clearance.
    - `useAppendAffiliateIds` sends batch affiliate payments.
- **Helper modules:**
    - `useFetchData` for paginated, sorted data fetching.
    - `AffiliatesSortModel` to handle grid sorting logic.
- **Service:** fetchAllGeneralAffliatesData calls the backend API.
- **Routes and columns** config imported for reuse.

## Chunk 2: State Setup
```typescript
const [selectedBooking, setSelectedBooking] = useState(0)
const [selectedAffiliates, setSelectedAffiliates] = useState<number[]>([])
const [searchFieldText, setSearchFieldText] = useState('')
const [clearanceStatusId, setClearanceStatusId] = useState(0)
const [clearanceDate, setClearanceDate] = useState('')
const [amount, setAmount] = useState(0)
const [isExcluxdedAffliates, setIsExcludedAffiliates] = useState(false)
const [isExcludeAffiliatesWithExpiredIds, setIsExcludeAffiliatesWithExpiredIds] = useState(false)
```

**Explanation:**
- **selectedBooking:** Filter by booking category.
- **selectedAffiliates:** IDs of affiliates selected via checkboxes.
- **searchFieldText:** Text search for affiliate names.
- **clearanceStatusId, clearanceDate, amount:** Track the current clearance record's filters.
- **isExcludedAffiliates, isExcludeAffiliatesWithExpiredIds:** Toggle exclusion filters.

## Chunk 3: Fetching Clearance Status & Handling Lock/Complete
```typescript
const { data: clearanceData, refetch: fetchStatus } = useClearanceStatus()
const { errors, disabled, handleClearance, clearanceDisable } = useAppendClearanceData({
  affiliates_more_than_euro: amount,
  clearance_date: clearanceDate,
  id: clearanceData?.id,
  fetchStatus
})
```

**Explanation:**
- `useClearanceStatus` retrieves the latest clearance record (date, threshold, id).
- `useAppendClearanceData` exposes:
    - `handleClearance(mode)` to lock or complete clearance.
    - `disabled` & `clearanceDisable` flags control button states.
    - `errors` for invalid inputs.

## Chunk 4: Fetching Unpaid Affiliates Data
```typescript
const {
  data: unpaidAffiliates,
  fetchData: fetchAllUnpaidAfilliatesData,
  loading,
  totalRows,
  paginationModel,
  setPaginationModel,
  setSortColumn,
  setSortType,
  sortColumn,
  sortType
} = useFetchData(fetchAllGeneralAffliatesData, AFFILIATES_URLS.UNPAID_PER_AFFILIATE, {
  is_exclude_fake_affiliates: isExcluxdedAffliates,
  is_exclude_expired_affiliates: isExcludeAffiliatesWithExpiredIds,
  ...(clearanceStatusId ? { clearance_status_id: clearanceStatusId } : {}),
  ...(clearanceDate ? { clearance_date: clearanceDate } : {}),
  affiliates_more_than_euro: amount,
  booking_category: selectedBooking === 0 ? 1 : selectedBooking
})
```

:**Explanation:**
- Uses `useFetchData` to call fetchAllGeneralAffliatesData with filter params.
- Returns data array, loading flag, total count, pagination & sorting state.

## Chunk 5: Handling Batch Payment

```typescript
const { handleAffiliatePayments, disabled: disablePaymentsField } = useAppendAffiliateIds({
  selectedAffiliates,
  fetchAllUnpaidAfilliatesData
})
```

**Explanation:**
- `handleAffiliatePayments(mode)` triggers API call for selected affiliate IDs.
- `disablePaymentsField` controls payment buttons when no selection

## Chunk 6: Syncing Clearance Data with Local State

```typescript
  setClearanceDate(clearanceData?.clearance_date?.toString() || '')
  setClearanceStatusId(clearanceData?.id ?? 0)
  setAmount(clearanceData?.affiliates_more_than_euro ?? 0)
}, [clearanceData])
```

**Explanation:**
- Updates local state when `clearanceData` changes.
- Initializes filters on mount or after lock/complete.

## Chunk 7: Filter Handler

```typescript
const handleFilter = <T,>(value: T, mode: MODE): void => {
  switch (mode) {
    case 'BOOKINGS': setSelectedBooking((value as any).target.value as number); break
    case 'EXCLUDED': setIsExcludedAffiliates(value as boolean); break
    case 'EXPIRED': setIsExcludeAffiliatesWithExpiredIds(value as boolean); break
    case 'CLEARNCE': setClearanceDate(value as string); break
    case 'SEARCH': setSearchFieldText(value as string); break
    case 'AMOUNT': setAmount((value as any).target.value as number); break
    default: toast('Something went wrong')
  }
}
```

**Explanation:**
- Central handler for all filter UI changes, updating relevant state.

## Chunk 8: Auto-fetch on Filter / Pagination / Sort Change

```typescript
useEffect(() => {
  if (!clearanceDate || !clearanceStatusId) return
  fetchAllUnpaidAfilliatesData()
}, [
  paginationModel,
  selectedBooking,
  isExcluxdedAffliates,
  isExcludeAffiliatesWithExpiredIds,
  clearanceDate,
  amount,
  clearanceData,
  clearanceStatusId
])
```

**Explanation:**
- Triggers data fetch whenever any filter, pagination, sort, or clearance record changes.
- Ensures grid data is always up-to-date.

## Chunk 9: Row Selection Handler

```typescript
const handleSelectedAffiliates = (checked: boolean, id: number) => {
  setSelectedAffiliates(prev =>
    checked ? [...prev, id] : prev.filter(prevId => prevId !== id)
  )
}
```

**Explanation:**
- Adds/removes affiliate IDs in the `selectedAffiliates` array when checkboxes toggle.

## Chunk 10: Column Definitions

Columns are defined in `UnpaidAffiliatesColumns`, including:
- Checkbox selector
- Affiliate name linking to details
- Contract type, unpaid counts, amounts
- Expiration dates highlighted if past
- Links/tooltips for email history and invoice info
- **statusDropdown** to update individual clearance status

## Chunk 11: JSX Render

```typescript
return (
  <Grid container spacing={2.5}>
    <AffiliatesUnpaidFilters ... />
    <Grid item xs={12}>
      <DatagridScrollWrapper>
        <MUIDataGrid
          columns={columns}
          rows={unpaidAffiliates}
          loading={loading}
          rowCount={totalRows}
          paginationModel={paginationModel}
          onPaginationModelChange={setPaginationModel}
          onSortModelChange={handleSortModel}
        />
      </DatagridScrollWrapper>
    </Grid>
  </Grid>
)
```

**Explanation:**
- **Filters toolbar:** `AffiliatesUnpaidFilters` handles all filter and action buttons.
- **Data grid:** wrapped for scroll behavior, with pagination, sorting, and custom columns.

## Error Handling
- **Validation** errors from useAppendClearanceData show inline on date/amount inputs.
- **Toast notifications** inform the user of unexpected errors in filter handler.

## Return Value
This component returns JSX elements: a filters toolbar and a data grid. It does not return any other value.

---

## Key Features
- **Clearance Workflow Controls:** Lock & complete clearance with clear UI states.
- **Dynamic Filters:** Filter by cutoff date, minimum unpaid amount, booking category, excluded affiliates, and expired IDs.
- **Batch Actions:** Proceed to payment and pay checked affiliates in bulk.
- **Individual Status Updates:** Status dropdown per affiliate for fine-grained clearance management.
- **Real-time Data:** Automatic data fetching with pagination and sorting based on active filters.
- **Visual Cues:** Locked fields are grayed out; expired dates highlighted in error color.
- **Interactive DataGrid:** Responsive MUI DataGrid wrapped for scroll behavior and custom column definitions.
- **Error Handling & Notifications:** Inline validation errors and toast alerts for feedback.