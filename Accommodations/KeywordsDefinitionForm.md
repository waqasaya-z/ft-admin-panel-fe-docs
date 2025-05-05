# Keywords definitions
***Micro-service | Fg-Admin-Panel-Frontend | Accommodations Module***

## 1. Component Name
**KeywordsDefinitionForm** — This component allows an admin to define and manage keyword logic rules (AND/OR conditions) for accommodation grouping under the settings section.

## 2. Overall Description
This component renders a dynamic form that enables the admin to set logical keyword combinations used to classify or group accommodations. It supports adding up to five keyword-condition pairs and saves them to the backend using `upsertAccommodationData`. The keywords are fetched from a central data table via the `useEssentialData` hook.

---

## 3. Breakdown of the Component


## Chunk 1: Import Statements
```tsx
import { Box, Button, Grid, IconButton, MenuItem, Tooltip } from '@mui/material'
import { HttpStatusCode } from 'axios'
import React, { useEffect } from 'react'
import { Controller, useForm } from 'react-hook-form'
import toast from 'react-hot-toast'
import Icon from 'src/@core/components/icon'
import CustomTextField from 'src/@core/components/mui/text-field'
import { updateAccommodationData, upsertAccommodationData } from 'src/api/accommodations.service'
import { TABLES } from 'src/constants/common.constants'
import useEssentialData from 'src/hooks/useEssentialData'
import { GroupKeywordDefinitionT } from 'src/types/accommodations.types'
import { AC_URL } from '../../utils/common.constants'
```

**Explanation:**
This block brings in all the required modules:
- `MUI components` for layout and interaction
- `react-hook-form` for form handling
- toast for notifications
- API functions and constants for data operations
- A custom hook `useEssentialData` to retrieve keyword options

## Chunk 2: Component Declaration and Hook Initialization
```tsx
const KeywordsDefinitionForm = ({ id, data }: { id: number; data: GroupKeywordDefinitionT[] }) => {
  const {
    data: keywords,
    loading: keywordLoading,
    error: keywordError
  } = useEssentialData(TABLES.ACCOMMODATIONS_KEYWORDS, [])
  const [dropdowns, setDropdowns] = React.useState<number[]>([0])
  const { control, handleSubmit, unregister, setValue, getValues, reset } = useForm()
```

**Explanation:**
- `id` and `data` are props representing the accommodation group and existing keyword definitions.
- Initializes dropdown index state.
- Uses `useForm` to manage form controls dynamically.
- Fetches keyword options via `useEssentialData`.

## Chunk 3: Hydrating the Form from Props

```tsx
useEffect(() => {
  if (data.length > 0) {
    setDropdowns(data.map((_, index) => index))
    data.forEach((item, index) => {
      setValue(`logical_operator_${index}`, item.logical_operator || 0)
      setValue(`keyword_id_${index}`, item.keyword_id || 0)
    })
  } else {
    setDropdowns([0])
  }
}, [data, setValue])
```

**Explanation:**
On mount or when data changes:
- Initializes form fields and dropdown indices based on existing data.
- Sets default values in the form for each keyword condition.

## Chunk 4: Adding and Removing Dropdowns
```tsx
const addDropdown = () => {
  if (dropdowns.length >= 5) return
  setDropdowns(prev => [...prev, prev.length])
  const newIndex = dropdowns.length
  setValue(`condition_${newIndex}`, 0)
  setValue(`keywords_${newIndex}`, 0)
}

const removeDropdown = (index: number) => {
  const currentValues = getValues()
  unregister(`condition_${index}`)
  unregister(`keywords_${index}`)
  setDropdowns(prev => {
    const updated = prev.filter((_, i) => i !== index)
    updated.forEach((_, newIndex) => {
      if (newIndex >= index) {
        setValue(`condition_${newIndex}`, currentValues[`condition_${newIndex + 1}`] || 0)
        setValue(`keywords_${newIndex}`, currentValues[`keywords_${newIndex + 1}`] || 0)
      }
    })
    return updated
  })
}
```

**Explanation:**
- `addDropdown`: Adds a new pair of dropdowns, limited to 5 max.
- `removeDropdown`: Removes a dropdown at a specified index and shifts the remaining ones to avoid gaps.


## Chunk 5: Submission Logic
tsx
const onSubmit = async (formData: any) => {
  const payload = dropdowns.map((_, index) => ({
    group_id: id,
    logical_operator: formData[`logical_operator_${index}`],
    keyword_id: formData[`keyword_id_${index}`]
  }))

  try {
    const response = await upsertAccommodationData(payload, AC_URL.KEYWORDS_DEFINITION)
    if (response.status === HttpStatusCode.Ok) {
      toast.success('Data was updated')
    } else {
      toast.error('Something went wrong')
    }
  } catch (e) {
    toast.error('Something went wrong', (e as any).message)
  }
}

**Explanation:**
- Transforms form data into the correct backend format.
- Submits via `upsertAccommodationData.`
- Handles both success and failure using toast notifications.

## Chunk 6: Rendered JSX (Form UI)

```tsx
return (
  <>
    <Box sx={{ mb: '20px', display: 'flex' }}>
      <Tooltip arrow placement='top' title={dropdowns.length >= 5 ? 'Maximum Limit Reached' : ''}>
        <Box>
          <Button variant='contained' onClick={() => addDropdown()} disabled={dropdowns.length >= 5}>
            <Icon fontSize='1.125rem' icon='tabler:plus' />
            ADD NEW
          </Button>
        </Box>
      </Tooltip>
    </Box>
    <form onSubmit={handleSubmit(onSubmit)}>
      {dropdowns.map((_, index) => (
        <Grid key={index} container spacing={3} sx={{ mb: '20px' }}>
          <Grid item>
            <Controller
              name={`logical_operator_${index}`}
              control={control}
              render={({ field: { value, onChange } }) => (
                <CustomTextField select className='custom-dropdown' value={value || 0} onChange={onChange}>
                  <MenuItem value={0}>ALL</MenuItem>
                  <MenuItem value={1}>OR</MenuItem>
                  <MenuItem value={2}>AND</MenuItem>
                </CustomTextField>
              )}
            />
          </Grid>
          <Grid item>
            <Controller
              name={`keyword_id_${index}`}
              control={control}
              render={({ field: { value, onChange } }) => (
                <CustomTextField select className='custom-dropdown' value={value || 0} onChange={onChange}>
                  <MenuItem value={0}>ALL</MenuItem>
                  {keywords.map((keyword: any) => (
                    <MenuItem key={Number(keyword.id)} value={Number(keyword.id)}>
                      {String(keyword.name)}
                    </MenuItem>
                  ))}
                </CustomTextField>
              )}
            />
          </Grid>
          <Grid item>
            <Tooltip title='Delete'>
              <IconButton onClick={() => removeDropdown(index)} disabled={dropdowns.length === 1}>
                <Icon icon='material-symbols:delete-outline' />
              </IconButton>
            </Tooltip>
          </Grid>
        </Grid>
      ))}
      <Box sx={{ mt: 2, display: 'flex' }}>
        <Button type='submit' variant='contained' color='primary'>
          Save
        </Button>
      </Box>
    </form>
  </>
)
```

**Explanation:**
- Dynamically renders a list of dropdowns for each logical condition and keyword pair.
- The user can add up to 5 rules.
- A delete button is available for each rule unless it's the last one.
- The form submits via `handleSubmit(onSubmit)`.


---

## 4. Return Value / Responses
There is no return value to the component itself. On form submission:
- A toast is shown to indicate success or failure.
- The form does not reset after submission.

## 5. Error Handling
If an API call fails, an error toast with the message is shown.
- Dropdowns cannot be added beyond 5; the button gets disabled and a tooltip indicates the limit.
- Deletion of the last remaining dropdown is prevented by disabling the delete button.


---
## Key Features
- Dynamically add/remove keyword conditions (up to 5)
- Form initialization with existing backend data
- Use of MUI components for a consistent UI
- Form state and validation using react-hook-form
- Tooltip-based feedback for max-limit & delete actions
- Persists changes via upsert API call
- Displays feedback via toast notifications
- Prevents deletion when only one field remains