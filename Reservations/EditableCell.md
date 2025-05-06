# Editable Cell (function)
***Micro-service |  Fg-Admin-Panel-Frontend | New Reservations Module*** 

## 1. Function Name
`EditableCell` — A render‑cell helper for MUI DataGrid that provides inline editing of any cell, with icons to save or cancel edits, and discarding unsaved changes when clicking outside.

## 2. Overall Description
`EditableCell` is intended to be passed into a column’s `renderCell` in an MUI DataGrid. It shows either a read‑only value with an edit icon, or, once activated, the appropriate input control (text, dropdown, date picker, or checkbox) alongside save/cancel icons. Unsaved edits are automatically rolled back if the user clicks anywhere outside the cell. Final changes across the grid can then be persisted via a parent-level submit button.

---
## 3. Breakdown of the Component

### Chunk 1: Import Statements & Props

```tsx
import React, { useRef, useEffect } from 'react'
import { GridCellModesModelProps } from '@mui/x-data-grid'
import { Box, Select, MenuItem, Checkbox } from '@mui/material'
import DateInputField from 'src/components/common/DateInputField'
import CustomTextField from 'src/components/common/CustomTextField'
import DoneIcon from '@mui/icons-material/Done'
import EditIcon from '@mui/icons-material/Edit'
import CancelIcon from '@mui/icons-material/Close'
```

**Explanation:**
- Core React hooks for refs and effects.
- MUI components for layout and controls.
- Custom inputs (text, date picker).
- Icon components for edit, save, and cancel.
- Props include cell params, cell mode state, handlers for edit/save/cancel, and arrays specifying which fields render as text, dropdown, date, or checkbox.

### Chunk 2: Determining Edit Mode & Outside Click Logic

```tsx
const { id, field, value } = params
const cellRef = useRef<HTMLDivElement>(null)

const isInEditMode =
  (cellModesModel[`${id}-${field}`] as unknown as GridCellModesModelProps)?.mode === 'edit'

useEffect(() => {
  if (!isInEditMode || !cellRef.current) return
  const handleClickOutside = (event: MouseEvent) => {
    const path = event.composedPath()
    const isInside = path.some(el => cellRef.current?.contains(el as Node))
    if (!isInside) handleCancelClick(id, field)()
  }
  document.addEventListener('mousedown', handleClickOutside)
  return () => document.removeEventListener('mousedown', handleClickOutside)
}, [isInEditMode, cellRef, handleCancelClick, id, field])
```

**Explanation:**
- Uses a ref on the cell wrapper to detect clicks outside.
- When in edit mode, attaches a document-level listener that cancels edits if the click happened outside the cell or any popover components.
- Cleans up the listener when unmounting or exiting edit mode.

### Chunk 3: Determining Input Type & Display Value

```tsx
const isText = textFields.includes(field)
const isDropdown = dropdownFields.includes(field)
const isDate = calendarFields.includes(field)
const isCheckbox = checkboxFields.includes(field)
const isDisabled = disabledFields.includes(field)

let displayValue = value
if (isDropdown && dropdownValues[field]) {
  displayValue = dropdownValues[field].find(opt => opt.value === value) || value
}
```

**Explanation:**
- Checks arrays of field names to choose the correct input component.
- For dropdowns, looks up the full option object to render labels.

## Chunk 4: Render Edit Mode vs. Read Mode

```tsx
return isInEditMode ? (
  <Box ref={cellRef} sx={{ display: 'flex', alignItems: 'center' }}>
    {isText && (
      <CustomTextField
        value={value}
        onChange={e => handleFieldChange(id, field, e.target.value)}
      />
    )}
    {isDropdown && (
      <Select
        value={displayValue.value}
        onChange={e => handleFieldChange(id, field, e.target.value)}
      >
        {dropdownValues[field].map(opt => (
          <MenuItem key={opt.id} value={opt.value}>{opt.label}</MenuItem>
        ))}
      </Select>
    )}
    {isDate && (
      <DateInputField
        value={value}
        onChange={date => handleFieldChange(id, field, date)}
      />
    )}
    {isCheckbox && (
      <Checkbox
        checked={!!value}
        onChange={e => handleFieldChange(id, field, e.target.checked)}
      />
    )}
    <DoneIcon onClick={handleSaveClick(params)} />
    <CancelIcon onClick={handleCancelClick(id, field)} />
  </Box>
) : (
  <Box sx={{ display: 'flex', alignItems: 'center' }}>
    <span>{String(displayValue)}</span>
    {isEdit && !isDisabled && <EditIcon onClick={handleEditClick(id, field)} />}
  </Box>
)
```

**Explanation:**
- **Edit Mode:** Renders the chosen input component, wired to `handleFieldChange`, plus save/cancel icons.
- **Read Mode:** Displays the formatted value and an edit icon (if editing is allowed) that invokes `handleEditClick`.

---

## 4. Return Value

The component returns JSX which conditionally renders:

- A flex container with an input (text, dropdown, date, or checkbox) and **Done/Cancel** icons in edit mode.
- A flex container with the cell’s value and an **Edit** icon in read mode.

This JSX integrates directly into the DataGrid’s cell rendering pipeline.

## 5. Error Handling & Edge Cases
- **Outside Click:** Automatically cancels edits if the user clicks outside the active cell or any associated popover.
- **Field Restrictions:** Certain fields (e.g. refund_percentage when status isn’t Cancelled) can be disabled or rendered read-only.
- **Invalid Values:** If a dropdown lookup fails, falls back to displaying the raw value.
- **Disabled Fields:** Hides the edit icon and prevents entering edit mode for fields marked disabled.

---

## Key Features
1. **Inline Cell Editing**
Allows direct editing within MUI DataGrid cells using various input types.

2. **Multi-Type Input Support**
Supports text fields, dropdowns, date pickers, and checkboxes, determined dynamically by field name.

3. **Intelligent Mode Switching**
Renders in read-only mode by default, switching to edit mode with an icon click.

4. **Auto-Cancel on Outside Click**
Detects clicks outside the cell and cancels unsaved edits automatically.

5. **Edit Icons**
Shows Edit, Save, and Cancel icons for intuitive editing interactions.

6. **Configurable Field Behavior**
Supports disabling certain fields, preventing editing based on custom rules.

7. **Dropdown Label Resolution**
Converts stored dropdown values into readable labels using mapped options.

8. **Reusable & Declarative Design**
Works as a plug-and-play renderCell function, simplifying integration.

9. **Custom Component Integration**
Easily hooks into your own input components like DateInputField or CustomTextField.

10. **Graceful Fallbacks**
If value mapping fails (e.g., invalid dropdown value), displays raw data instead of crashing.

