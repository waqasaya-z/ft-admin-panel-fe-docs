# List Our Reservations (Reservations Listing)
***Micro-service |  Fg-Admin-Panel-Frontend | New Reservations Module*** 

## 1. Component Name
`ListOurReservations.tsx` — The main listing component for the Reservations module in the admin panel of a ferry ticket website. It displays all reservations made on the website with filtering, searching, and pagination functionality.

## 2. Overall Description
`ListOurReservations` is rendered when the admin clicks on New Reservations in the sidebar. It serves as the entry point to view and manage every booking made through the ferry ticket website. The component provides:

- **Filter controls** for quick segmentation (failed bookings, status, travel date/status).
- **Search capability** to look up bookings by any displayed field.
- **Export options** via an Excel download menu (today, last week, last month, custom range).
- A **hierarchical data grid** (tree structure) showing parent reservations expandable into individual route details.
- An "**Add Admin Notes**" modal for annotating any reservation.

This component ties together API calls, state management, and UI controls to give administrators a comprehensive and interactive overview of reservations.

---

## 3. Breakdown of the Component

### Chunk 1: Import Statements
```tsx
import React, { useCallback, useEffect, useState } from 'react'
import { GridPaginationModel, GridSortModel } from '@mui/x-data-grid'
import { Grid, Tooltip, IconButton, debounce, Box } from '@mui/material'
import MUIDataGrid from 'src/@core/components/mui/datagrid'
import { FiltersT, ReservationT } from 'src/types/reservations.types'
import {
  fetchAllCustomerReservationsWithPagination,
  fetchAllReservationsWithPagination,
  patchNotes
} from 'src/api/reservations.service'
import DatagridScrollWrapper from 'src/components/common/feature/DatagridScrollWrapper'
import Icon from 'src/@core/components/icon'
import { ReservationColumns } from './columns'
import ReservationSearchField from './ReservationSearchField'
import ReservationFilterList from './ReservationFilterList'
import { useRecoilValue } from 'recoil'
import { customerState } from 'src/recoil/atoms/atoms'
import ResponsiveDialog from 'src/components/common/base/ResponsiveDialog'
import AdminNotesForm from 'src/components/reservations/cancelled-reservations/AdminNotesForm'
```

**Explanation:**
- Core React hooks (`useState`, `useEffect`, `useCallback`) manage component lifecycle and state.
- Material UI imports (`Grid`, `IconButton`, etc.) for layout and controls.
- `MUIDataGrid` and `DatagridScrollWrapper` provide the data table UI with scrolling support.
- Type definitions `FiltersT` and `ReservationT` enforce filter query shape and reservation data shape.
- API services for fetching reservations (all vs. per-customer) and patching admin notes.
- Custom components for search input, filter list controls, responsive dialogs, and notes form.
- Recoil state to conditionally fetch customer-specific reservations if `leader_id` is present.

### Chunk 2: Component State Setup
```tsx
const [paginationModel, setPaginationModel] = useState({ page: 0, pageSize: 10 })
const [totalRows, setTotalRows] = useState<number>()
const [failedBookingsActive, setFailedBookingsActive] = useState(false)
const [travellingTodayActive, setTravellingTodayActive] = useState(false)
const [reservations, setReservations] = useState<ReservationT[]>([])
const [expandedRows, setExpandedRows] = useState<Record<string | number, boolean>>({})
const [searchText, setSearchText] = useState('')
const [openModal, setOpenModal] = useState(false)
const [adminNoteId, setAdminNotesId] = useState<string | null>(null)
const [filters, setFilters] = useState<FiltersT>({
  page: 1,
  limit: 10,
  sort_type: 'desc',
  sort_column: 'departure_date',
  status: null,
  departure_date_from: '',
  departure_date_to: '',
  searched_text: null,
  traveling_status: null
})
```

**Explanation:**
- `paginationModel` & `totalRows` track current page, page size, and total records for pagination.
- Boolean flags (`failedBookingsActive`, `travellingTodayActive`) toggle quick filters.
- `reservations` stores the fetched data array.
- `expandedRows` keeps track of which parent rows are expanded to reveal child (route) entries.
- `searchText` reflects the raw input in the search bar; debounced updates drive backend queries.
- Modal state (`openModal`, `adminNoteId`) controls the visibility and context of the "Add Admin Notes" dialog.
- `filters` holds the full query payload sent to the API, including sorting, status, date ranges, and text search.

### Chunk 3: Filter Utilities

```tsx
const handleFilterChange = (newFilter: Partial<FiltersT>) => {
  setFilters(prev => ({ ...prev, ...newFilter }))
}

const generateFilterQuery = (filters: FiltersT) => {
  const query = { ...initialFilterSkeleton }
  for (const key in filters) {
    if (filters[key] !== null && filters[key] !== '') {
      query[key] = filters[key]
    }
  }
  return query
}

const handleSearchChange = debounce((value: string) => {
  handleFilterChange({ searched_text: value || null })
}, 500)
```

**Explanation:**
- `handleFilterChange` merges incoming filter updates into the existing state.
- `generateFilterQuery` constructs a query object, omitting null or empty fields to minimize API payload.
- The debounced `handleSearchChange` delays filter updates until the user stops typing (500ms), preventing excessive API calls.

### Chunk 4: Data Fetching

```tsx
const fetchReservations = useCallback(async () => {
  const query = generateFilterQuery(filters)
  setLoading(true)
  try {
    let response
    if (leader_id) {
      response = await fetchAllCustomerReservationsWithPagination(query, leader_id, email)
    } else {
      response = await fetchAllReservationsWithPagination(query)
    }
    if (response.status === HttpStatusCode.Ok) {
      setReservations(response.data.data.data)
      setTotalRows(response.data.data.total)
    } else {
      setReservations([])
      setTotalRows(0)
    }
  } catch {
    setReservations([])
    setTotalRows(0)
  } finally {
    setLoading(false)
  }
}, [filters, leader_id, email])

useEffect(() => {
  fetchReservations()
}, [paginationModel.page, paginationModel.pageSize, fetchReservations])
```

**Explanation:**
- `fetchReservations` is wrapped in useCallback to memoize based on `filters`, `leader_id`, and `email`.
- It toggles the `loading` flag, calls the appropriate API endpoint, and updates `reservations` and `totalRows` on success.
- Error handling clears the data arrays to avoid stale UI.
- A `useEffect` hook triggers `fetchReservations` whenever pagination or filters change.

### Chunk 5: Expand/Collapse & Modal Handlers

```tsx
const handleExpandClick = (id: number) => { ... }
const handleModalOpen = (id: string) => { ... }
const onClose = () => { ... }
```

**Explanation:**
- `handleExpandClick` toggles a row’s expanded state, adding or removing it from `expandedRows`.
- `handleModalOpen` and `onClose` manage opening and closing of the "Add Admin Notes" dialog, storing the target reservation ID.

### Chunk 6: Column Definitions & Row Flattening

```tsx
const parentIds = new Set(
  reservations.filter(r => r.booking_details.length > 0).map(r => r.id)
)

const columns = [
  { field: 'expand', headerName: 'Booking Number', renderCell: ... },
  { field: 'details', headerName: 'Show Details', renderCell: ... },
  { field: 'itinerary_count', headerName: 'Itinerary #', renderCell: ... },
  ...ReservationColumns({ parentIds, handleModalOpen })
].map(col => ({ ...col, sortable: false }))

const visibleRows = reservations.flatMap(row => {
  if (expandedRows[row.id]) {
    return [row, ...row.booking_details.map(...)]
  }
  return [row]
})
```

**Explanation:**
- `parentIds` identifies rows with child route entries.
- The `columns` array defines custom renderers for expansion toggles, detail links, itinerary counts, and other fields imported from `ReservationColumns`.
- `visibleRows` flattens parent and child rows based on `expandedRows` state to feed into the grid.

### Chunk 7: JSX Rendering

```tsx
return (
  <>
    {/* Filter List & Search */}
    <Grid container spacing={2}>
      <ReservationFilterList ... />
      <ReservationSearchField ... />
    </Grid>

    {/* Data Grid */}
    <DatagridScrollWrapper>
      <MUIDataGrid
        rows={visibleRows}
        columns={columns}
        rowCount={totalRows}
        loading={loading}
        paginationModel={paginationModel}
        onPaginationModelChange={...}
        onSortModelChange={...}
        getRowHeight={...}
        getRowClassName={...}
        disableRowSelectionOnClick
      />
    </DatagridScrollWrapper>

    {/* Admin Notes Modal */}
    <ResponsiveDialog open={openModal} onClose={onClose} title="Add Admin Notes">
      <AdminNotesForm id={adminNoteId} submitHandler={patchNotes} ... />
    </ResponsiveDialog>
  </>
)
```

**Explanation:**
- Renders filter controls and search bar in a Material UI `Grid`.
- Wraps the data table in a scroll wrapper for better UX with large datasets.
- Configures the MUIDataGrid with pagination, sorting, and dynamic row rendering.
- Includes the admin notes modal at the root level so it can overlay the grid when triggered.

---

## 4. Return Value
The component returns JSX fragments comprising:

- A **filter bar** (buttons, dropdowns, search field).
- A **download Excel** dropdown (handled inside ReservationFilterList).
- A **hierarchical data grid** (MUIDataGrid) showing reservations and child routes.
- A **modal dialog** (`ResponsiveDialog` + `AdminNotesForm`) for adding notes.
All these pieces combine to render the complete interactive reservations list UI.

## 5. Error Handling
- **API Errors:** `fetchReservations` wraps calls in a `try/catch`. On errors, it clears `reservations` and `totalRows` to avoid displaying stale data and stops the loading spinner.
- **Empty States:** If the API returns an error status or an empty data array, the grid shows no rows and pagination reflects zero results.
- **Network Issues:** The `catch` block handles network or parsing errors gracefully by resetting the state.
- **User Feedback:** The `loading` prop on `MUIDataGrid` shows a spinner while fetching. Any failures result in an empty grid rather than a crash.

---

## Key features
- **Hierarchical Data Grid:** Displays parent reservation entries with expandable child rows showing individual route details.
- **Live Search:** Search functionality with debounce to prevent excessive API calls and ensure smooth filtering.
- **Dynamic Filtering:** Multiple filters including travel status, departure date, and booking status. Toggle-based filters for "Failed Bookings" and "Travelling Today."
- **Pagination and Sorting:** Supports server-side pagination and sorting with real-time data updates from the backend.
- **Export to Excel:** Admins can download reservation data filtered by date ranges (today, week, month, custom).
- **Admin Notes Modal:** A responsive dialog that allows admins to add custom notes to any reservation.
- **Contextual Data Loading:** Conditionally loads data per customer if a `leader_id` exists, supporting sub-admin views.
- **Expand/Collapse Rows:** Parent reservations can be expanded to view child route-level booking details in-line.
- **Modular Architecture:** Well-structured and reusable code with clear separation of concerns (columns, search field, filter list, modal form).

