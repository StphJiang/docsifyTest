# task_ctrl API Reference


## `taskpush(taskname, targ)`

### PARAMETERS
| Parameter | Type | Description |
| :--- | :--- | :--- |
|**`taskname`**|`TASK_f`|Function pointer to task|
|**`targ`**|`void*`|void Pointer to task arguemnt|

### RETURN VALUE
| Type | Description |
| :--- | :--- |
| **`int`**|On success, zero is returned. On error, -1 is returned|

### DESCRIPTION
This function support an approach to push task to one thread in threadpool.

### SEE ALSO
[`parser_input()`](parse_input.md)
