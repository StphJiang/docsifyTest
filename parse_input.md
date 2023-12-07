# parse_input API Reference


## `parser_input(argc, argv)`

### PARAMETERS
| Parameter | Type | Description |
| :--- | :--- | :--- |
|**`argc`**|`int`|Count of arguements input from console|
|**`argv`**|`char**`|Pointer to arguement vector|

### RETURN VALUE
| Type | Description |
| :--- | :--- |
| **`int`**|On success, zero is returned. On error, -1 is returned|

### DESCRIPTION
Parse input arguements from console by shell, user can control the whole application by several arguements. More details can be shown by argument "-h" or "--help" for help.

### SEE ALSO
getopt(3), getopt_long(3)
[task_ctrl](task_ctrl.md)


## `is_log_debug()`

### PARAMETERS
| Parameter | Type | Description |
| :--- | :--- | :--- |
|**` `**|`void`|No param|

### RETURN VALUE
| Type | Description |
| :--- | :--- |
| **`bool`**|Return flags value user input, ture if arg "-d" is used by user|

### DESCRIPTION
Use this func to decide which log could show in debug mode while don't show any info as usual.

### SEE ALSO
[`is_log_verbose()`]()


## `is_log_verbose()`

### PARAMETERS
| Parameter | Type | Description |
| :--- | :--- | :--- |
|**``**|`void`|No param|

### RETURN VALUE
| Type | Description |
| :--- | :--- |
| **`bool`**|Return flags value user input, ture if arg "--verbose" is used by user|

### DESCRIPTION
Use this func to get more info in verbose mode. 

### SEE ALSO
[`is_log_debug()`]()


## `get_sensor_mode()`

### PARAMETERS
| Parameter | Type | Description |
| :--- | :--- | :--- |
|**``**|`void`|No param|


### RETURN VALUE
| Type | Description |
| :--- | :--- |
| **`char**`**|Get sensor mode input by user in console|

### DESCRIPTION
Get sensor mode string input from console by user, which can control sensor mode for different purpose. This is an addtional option function for specific purpose.

### SEE ALSO
