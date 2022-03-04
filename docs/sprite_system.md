# Sprite System
Explorers of Sky features several ways for displaying graphics to the screen; this document will focus on how the **sprite** approach is done, which sees use in displaying objects that may move around the screen or be animated in some way.
## Understanding the Hardware
To have a good understanding of how it all works, we first need to understand the way the hardware operates. Objects to be displayed on screen are all stored in **OAM (Object Attribute Memory)**, and each screen can feature at most 128 objects. OAM keeps track of the position of objects, what sprite they correspond to,
## RGB Adapter
The RGB adapter is a very common component in the graphics engine of this game, and is not unique to the sprite system, but it will be included here for now. The main purpose of this component is to buffer 8-bit RGB colors as input and write them elsewhere as 5-bit RGB. The adapter can operate on 6 different modes, some of these might apply transformations to the colors using a master weight value and/or individual weight values for each color.
### Adapter Modes
Here we'll be describing every individual component as x, every individual weight as w, and the master weight as mW. We will also work under the assumption that all of those values are in the range 0..255 (which should be always true save for the master weight, which is stored as an uint16_t). We also won't be including the conversion to 5-bit RGB, as that's simply taking the five most significant bits out of the resulting RGB component and shifting to wherever appropiate.
| Index | Explanation | Formula |
| --- | --- | --- |
| 0 | Keeps the components unchanged | x |
| 1 | Darkens each component as mW approaches zero | x * mW >> 8 |
| 2 | Darkens each component as mW approaches zero, with w as the minimum | x * mW + w * (0x100 - mW) >> 8 |
| 3 | Darkens the weights as mW approaches zero, set component as weights | w * mW >> 8 |
| 4 | Darkens the weights as mW approaches zero, with w - 255 as the minimum | x * (w + (0xff - w) * mW >> 8) / 0xff |
| 5 | Darkens the weights as mW approaches zero | x * (w * mW >> 8) / 0xff |

If the palette manager in charge of this adapter is located in the Extended OBJ Palette, then the 16 last colors out of every 256 colors are kept unchanged, without exceptions.
### Struct Definitions
struct rgb_adapter
| Offset | Type | Name | Description |
| --- | --- | --- | --- |
| 0x00 | palette_bufferer* | palette_bufferer | The palette bufferer in charge of this adapter, which owns the output buffer |
| 0x04 | uint32_t | color_amt | How many colors this adapter stores, as well as how many it writes to the output buffer |
| 0x08 | bool | updated | Whether this adapter received any changes in its palette and therefore should call the adapter function again |
| 0x0A | uint16_t | master_weight | A weight that is commonly used in calculations for all three components |
| 0x0C | uint8_t | r_weight | Weight applied to the R component as needed |
| 0x0D | uint8_t | g_weight | Weight applied to the G component as needed |
| 0x0E | uint8_t | b_weight | Weight applied to the B component as needed |
| 0x0F | uint8_t | x_weight | Unused weight, as result of how RGB colors are packed into 4 bytes |
| 0x10 | uint32_t | unused10 | Most likely unused |
| 0x14 | function* | adapter_func | Pointer to the adapter function that does the appropiate transformations and then writes the results to the 5-bit RGB buffer |
| 0x18 | rgb8* | input_palette | The palette that the adapter will apply transformations to before storing it in the 5-bit RGB buffer |
| 0x1C | rgb5* | output_buffer | The location to where to write the transformed packed 5-bit RGB components |
| 0x20 | void* | unused20 | Most likely unused, seems to be a pointer to an unknown struct |
| 0x24 | void* | unused24 | Most likely unused, seems to be a pointer to an unknown struct |
## Object Bufferer Sub-System
The object bufferer sub-system is one of the components of the Sprite System, which takes care not only of buffering writes to the OAM, but it also provides a cool functionality to sort objects by depth! The way it achieves this is by allocating a look-up table for the range of depths available, with each element Storing the index within the buffer of the last sprite to be stored at that depth. This table is then used to write the object data to the sorted buffer, which is finally copied to OAM. The table is read from last to first, which means the highest index on the table is placed closest to the start of OAM and therefore has higher priority. This sub-system doesn't touch the attribute data itself, including intrinsic priorities.
### Struct Definitions
struct object_bufferer (32 bytes)
| Offset | Type | Name | Description |
| --- | --- | --- | --- |
| 0x00 | uint32_t | buffer_capacity | How many objects the unsorted and unsorted buffer can hold |
| 0x04 | uint32_t | sort_map_size | How many different depths are allowed in this sub-system |
| 0x08 | uint32_t | object_amount | Counter of how many objects are stored in the unsorted buffer (and by extension how many will be stored in the sorted buffer), also used to incrementally assign indexes to each object that's stored |
| 0x0C | int16_t* | sort_map | Pointer to the sort map, where each value is the index of the last object stored at that particular index. A value of -1 represents no objects have been stored in that location |
| 0x10 | object_bufferer_entry* | unsorted_buffer | Pointed to the unsorted buffer, which is where elements are stored as they're added before being sorted |
| 0x14 | bool | should_copy | Whether the sorted buffer should be copied to OAM when the bufferer's respective method is called |
| 0x18 | objattr8* | oam_dest | Pointer to where to copy the objects inside the sorted buffer |
| 0x1C | objattr8* | sorted_buffer | Elements are copied from the unsorted buffer to this buffer during sorting. This is the buffer that gets directly copied to OAM. The fourth value that is used to configure rotation/scaling groups is unused and initialized to zero |

struct object_bufferer_entry (8 bytes)
| Offset | Type | Name | Description |
| --- | --- | --- | --- |
| 0x00 | objattr | obj_attr | The object attributes of this entry |
| 0x06 | int16_t | next_index | The index of the next entry, or -1 if this is the first entry to be added (or the last entry to be accessed, same thing) |
