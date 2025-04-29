# ServiceFee Component Documentation

## 1. Introduction to Component
This is a **basic listing component** used in the **Fees and Commissions** module. It follows a standard listing structure that is reused across multiple listing tabs in the application, ensuring consistency in UI and behavior.

## 2. Imports Explanation
**`useCallback`, `useEffect`, `useState`** – React hooks used for managing side effects and state.  
**`MUIDataGrid`** – Custom wrapper for Material UI Data Grid used for displaying data in table format.  
**`Box`, `Button`** – MUI components for layout and action controls.  
**`GridSortModel`** – Type definition for handling sort model in the data grid.  
**`Icon`** – Custom icon component.  
**`fetchAllServiceFee`** – API call to retrieve service fee data.  
**`ServiceFeeProps`** – Type definition for a service fee item.  
**`SortType`** – Enum/type for sorting order.  
**`HttpStatusCode`** – Axios enumeration for HTTP response statuses.  
**`ServiceFeeAddUpdateModal`** – Modal for adding or editing a service fee item.  
**`toast`** – Toast notification library for user feedback.  
**`hardDeleteEntity`** – Generic API function for deletion.  
**`TABLES`** – Constants for different database table names.  
**`ConfirmationDialog`** – Modal for confirming delete actions.  
**`useFeesAndCommissionContext`** – Context hook for accessing shared state.  
**`SortModel`** – Utility to handle sorting logic.  
**`FC_URL`** – Constants file for endpoint URLs.  
**`ServiceFeeColumn`** – Columns configuration for the data grid.  
**`DatagridScrollWrapper`** – Wrapper for scrollable datagrid content.

## 3. States and Context Used
```typescript
  const { siteDetails: selectedSiteId } = useFeesAndCommissionContext()
  const [totalRows, setTotalRows] = useState<number>()
  const [paginationModel, setPaginationModel] = useState({ page: 0, pageSize: 10 })
  const [loading, setLoading] = useState(true)
  const [isEdit, setIsEdit] = useState<number>(0)
  const [openModal, setOpenModal] = useState(false)
  const [sortColumn, setSortColumn] = useState<string>('id')
  const [sortType, setSortType] = useState<SortType>('desc')
  const [serviceFee, setServiceFee] = useState<ServiceFeeProps[]>([])
  const [deleteConfirmationModal, setDeleteConfirmationModal] = useState({ open: false, idToDelete: 0, entityName: '' })
```
**Local States:**
- **`totalRows`** – Holds the total number of rows for pagination.
- **`paginationModel`** – Manages the current page and page size.
- **`loading`** – Shows loader while fetching data.
- **`isEdit`** – Tracks the ID of the item being edited.
- **`openModal`** – Controls visibility of the add/edit modal.
- **`sortColumn`, `sortType`** – Stores current sorting configuration.
- **`serviceFee`** – Array of service fee data.
- **`deleteConfirmationModal`** – Object to handle delete confirmation modal state.

**From Context:**
- **`selectedSiteId`** – Retrieved from `useFeesAndCommissionContext` to fetch selected site id.

## 4. Functions Explained

```typescript
 const handleEditContent = (isEdit = 0) => {
    setIsEdit(isEdit)
    setOpenModal(true)
  }
``` 

### **`handleEditContent`**
**Purpose:** Opens the modal and sets the ID for editing.  
**Usage:** Called when user clicks edit.

```typescript
  const fetchAllServiceFeeData = useCallback(
    async (sort_type: SortType = sortType, sort_column = sortColumn) => {
      setLoading(true)
      try {
        const response = await fetchAllServiceFee(
          FC_URL.SERVICE_FEE,
          paginationModel.page + 1,
          paginationModel.pageSize,
          sort_type,
          sort_column,
          selectedSiteId
        )
        if (response.status === HttpStatusCode.Ok) {
          setServiceFee(response.data?.data?.data ?? [])
          setTotalRows(response.data?.data?.total)
        } else {
          setServiceFee([])
        }
      } catch (e) {
        setServiceFee([])
        toast.error('Data fetch failed. Please refresh the page to try again', (e as any).message)
      } finally {
        setLoading(false)
      }
    },
    [paginationModel, sortType, sortColumn, setServiceFee, selectedSiteId]
  )
``` 

### **`fetchAllServiceFeeData`**
**Purpose:** Fetches service fee data from the API with sorting and pagination parameters.  
**Handles:** Error feedback via toast, sets data and loading state.

```typescript
  useEffect(() => {
    fetchAllServiceFeeData()
  }, [paginationModel, selectedSiteId])
```

### **`useEffect`**
**Purpose:** Triggers data fetch on pagination or site change.  
**Dependency Array:** `paginationModel`, `selectedSiteId`.

```typescript
  const handleDelete = async (planId: number | undefined, entityName = '') => {
    try {
      if (!deleteConfirmationModal.open) {
        setDeleteConfirmationModal({ open: true, idToDelete: planId ?? 0, entityName: entityName })
        return true
      }
      setDeleteConfirmationModal({ ...deleteConfirmationModal, open: false })
      setLoading(true)
      const response = await hardDeleteEntity(TABLES.SERVICE_FEE, planId ?? 0)
      if (response.status == HttpStatusCode.Ok) {
        toast.success(response?.data?.message ?? '')
      } else {
        toast.error(response?.data?.message ?? '')
      }
    } catch (e: any) {
      setLoading(false)
      toast.error(e?.response?.data?.error ?? 'An unexpected error has occured')
    }
    fetchAllServiceFeeData()
  }
```

### **`handleDelete`**
**Purpose:** Deletes a service fee after user confirmation.  
**Includes:** 
- Modal trigger for confirmation
- API call to delete
- Toast for success/failure
- Refetch data after deletion

```typescript
const handleSortModel = (newModel: GridSortModel) =>
    SortModel(newModel, fetchAllServiceFeeData, sortType, sortColumn, setSortType, setSortColumn)
```

### **`handleSortModel`**
**Purpose:** Manages sorting of columns through a shared sorting utility.

```typescript
  const columns = ServiceFeeColumn({
    setLoading: setLoading,
    fetchFunc: fetchAllServiceFeeData,
    handleDelete: handleDelete,
    handleEdit: handleEditContent
  })
```
## 5. Columns Configuration
Columns are imported from a separate file:  
**`./columns`**  
This promotes reusability and keeps the main component clean.

```typescript
   <Box sx={{ mb: '20px', display: 'flex', alignItems: 'flex-end', justifyContent: 'flex-end' }}>
        <Button variant='contained' onClick={() => setOpenModal(true)}>
          <Icon fontSize='1.125rem' icon='tabler:plus' />
          ADD NEW
        </Button>
      </Box>
      <DatagridScrollWrapper>
        <MUIDataGrid
          rowCount={totalRows}
          columns={columns}
          rows={serviceFee}
          loading={loading}
          paginationModel={paginationModel}
          onPaginationModelChange={setPaginationModel}
          onSortModelChange={handleSortModel}
        />
      </DatagridScrollWrapper>
      <ServiceFeeAddUpdateModal
        openModal={openModal}
        setOpenModal={setOpenModal}
        isEdit={isEdit}
        setIsEdit={setIsEdit}
        fetchFunc={fetchAllServiceFeeData}
        data={serviceFee}
      />
      <ConfirmationDialog
        open={deleteConfirmationModal.open}
        onClose={() => setDeleteConfirmationModal({ idToDelete: 0, open: false, entityName: '' })}
        onConfirm={() => handleDelete(deleteConfirmationModal.idToDelete)}
        content={`Are you sure you want to delete "${deleteConfirmationModal.entityName}"?`}
      />
```

## 6. Component Usage Explained

### **`<Box>`**
**Use Case:** Wraps the "Add New" button with spacing and alignment.

### **`<Button>`**
**Use Case:** Triggers the modal to add a new service fee.

### **`<DatagridScrollWrapper>`**
**Use Case:** Ensures the data grid scrolls properly within layout constraints.

### **`<MUIDataGrid>`**
**Use Case:** Displays the service fee data in tabular format with pagination and sorting.

### **`<ServiceFeeAddUpdateModal>`**
**Use Case:** Modal form for adding or updating a service fee.

### **`<ConfirmationDialog>`**
**Use Case:** Confirms the user wants to delete a service fee before performing the action.
