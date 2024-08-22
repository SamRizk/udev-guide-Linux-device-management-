```python
# MIT License
#
# Copyright (c) 2024 Samir Rizk
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

# Author: Samir Rizk
# Date: 22/08/2024
# Copyright: 2024 Samir Rizk. All rights reserved.
```

# understanding and utilizing `udev` rules

### What is `udev`?

`udev` is a device manager for the Linux kernel that dynamically creates and removes device nodes in the `/dev` directory, handles user space events when devices are added or removed, and manages device permissions. It plays a crucial role in managing hardware devices by allowing you to define rules that specify how devices should be handled.

### How `udev` Works

When a device is added to the system, the kernel generates an event that `udev` processes. `udev` then applies a set of rules to determine how the device should be handled. This includes creating device nodes, setting permissions, and executing custom commands.

### `udev` Rules

`udev` rules are stored in files located in `/etc/udev/rules.d/`, `/lib/udev/rules.d/`, and `/run/udev/rules.d/`. The files in `/etc/udev/rules.d/` take precedence over the ones in `/lib/udev/rules.d/`. These rules are applied in lexical order based on the file name.

### Rule Structure

Each `udev` rule is a single line in a file and consists of key-value pairs. The general syntax of a rule is:

```
ACTION=="action", SUBSYSTEM=="subsystem", ATTR{attribute}=="value", [other keys] RUN+="/path/to/command"
```

- **ACTION**: The action that triggers the rule (e.g., `add`, `remove`, `change`, `online`, `offline`).
- **SUBSYSTEM**: The subsystem the rule applies to (e.g., `usb`, `net`, `block`).
- **ATTR{attribute}**: A specific attribute of the device (e.g., `ATTR{vendor}=="0x1234"`).
- **RUN**: A command to execute when the rule is matched.

### Common Key-Value Pairs

1. **ACTION**: The action performed on the device.
   - Example: `ACTION=="add"`

2. **KERNEL**: Matches the device name.
   - Example: `KERNEL=="sda"`

3. **SUBSYSTEM**: Matches the device's subsystem.
   - Example: `SUBSYSTEM=="usb"`

4. **ATTR{attribute}**: Matches a device attribute.
   - Example: `ATTR{idVendor}=="0x1234"`

5. **RUN**: Specifies a command to run when the rule is matched.
   - Example: `RUN+="/bin/sh -c 'echo 1 > /sys/class/leds/led0/brightness'"`

6. **NAME**: Sets the device node name.
   - Example: `NAME="my_device"`

7. **SYMLINK**: Creates a symbolic link to the device node.
   - Example: `SYMLINK+="my_device_link"`

8. **GROUP**: Sets the group ownership of the device node.
   - Example: `GROUP="audio"`

9. **MODE**: Sets the permissions of the device node.
   - Example: `MODE="0660"`

### Example `udev` Rules

#### 1. Creating a Custom Device Name

```bash
SUBSYSTEM=="block", KERNEL=="sda", NAME="my_custom_sda"
```

This rule renames the device `/dev/sda` to `/dev/my_custom_sda`.

#### 2. Creating a Symbolic Link

```bash
SUBSYSTEM=="usb", ATTR{idVendor}=="0x1234", ATTR{idProduct}=="0x5678", SYMLINK+="my_usb_device"
```

This rule creates a symbolic link `/dev/my_usb_device` to the USB device with the specified vendor and product ID.

#### 3. Setting Permissions and Group

```bash
SUBSYSTEM=="net", KERNEL=="eth0", GROUP="netadmin", MODE="0660"
```

This rule sets the group ownership of the network device `eth0` to `netadmin` and grants read and write permissions to the owner and group.

#### 4. Running a Command When a Device is Added

```bash
ACTION=="add", SUBSYSTEM=="usb", RUN+="/usr/bin/notify-send 'USB Device Connected'"
```

This rule runs the command to send a desktop notification when a USB device is connected.

### Advanced Usage

#### Matching Multiple Attributes

You can combine multiple attributes in a single rule to match more specific devices.

```bash
SUBSYSTEM=="usb", ATTR{idVendor}=="0x1234", ATTR{idProduct}=="0x5678", RUN+="/usr/bin/my_script.sh"
```

#### Matching by Environment Variables

You can use environment variables within the rule.

```bash
ACTION=="add", ENV{DEVTYPE}=="usb_device", RUN+="/usr/bin/my_usb_script.sh"
```

### Applying `udev` Rules

After creating or modifying `udev` rules, you need to reload the `udev` rules and trigger them for the changes to take effect:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

### Testing and Debugging

To test and debug `udev` rules, you can use the `udevadm` tool:

```bash
udevadm test /sys/class/tty/ttyUSB0
```

This command will simulate applying the rules to the specified device and output detailed information, including which rules were applied.

### Expected Output

The output of your `udev` rules depends on the commands you run and the actions you take. For example:

- **Custom Device Name**: A new device node with the specified name will appear in `/dev`.
- **Symbolic Link**: A symbolic link will be created pointing to the actual device node.
- **Permissions and Group**: The device node's permissions and group will be updated according to the rule.
- **Running a Command**: The specified command will be executed, and any output from that command will be generated.

### 1. Using `udevadm` to Inspect Devices

The `udevadm` command is a powerful tool for querying and debugging `udev` rules. You can use it to get detailed information about devices connected to your system.

#### Inspect a Specific Device

To get detailed information about a specific device, you can use:

```bash
udevadm info --query=all --name=/dev/sda
```

Replace `/dev/sda` with the device you want to inspect. This command will show all the attributes (`KERNEL`, `SUBSYSTEM`, `ATTR`, etc.) that you can use in your `udev` rules.

#### Example Output

The output of `udevadm info` might look something like this:

```bash
P: /devices/pci0000:00/0000:00:1f.2/ata1/host0/target0:0:0/0:0:0:0/block/sda
N: sda
S: disk/by-id/ata-Samsung_SSD
E: DEVLINKS=/dev/disk/by-id/ata-Samsung_SSD
E: DEVNAME=/dev/sda
E: DEVPATH=/devices/pci0000:00/0000:00:1f.2/ata1/host0/target0:0:0/0:0:0:0/block/sda
E: DEVTYPE=disk
E: ID_MODEL=Samsung_SSD
E: ID_SERIAL=1234567890
E: SUBSYSTEM=block
E: USEC_INITIALIZED=1234567
```

- **P**: Device path in the sysfs.
- **N**: Kernel name of the device.
- **S**: Symlink names.
- **E**: Environment variables associated with the device.

This output shows you all the attributes (`SUBSYSTEM`, `DEVPATH`, `DEVTYPE`, etc.) that you can use in your `udev` rules.

### 2. Listing All Subsystems

To list all available subsystems, you can use:

```bash
ls /sys/class/
```

This command lists all the subsystems present on your system, such as `usb`, `net`, `block`, etc.

### 3. Exploring Kernel Attributes (`ATTR`)

You can explore available attributes for a specific device by navigating through the `/sys` filesystem. For example:

```bash
ls /sys/class/block/sda/
```

This will list all the attributes available for the block device `sda`, which you can reference in `udev` rules using `ATTR{attribute_name}`.

### 4. Using `udevadm monitor`

To dynamically see events as they happen and view the `ACTION`, `KERNEL`, `SUBSYSTEM`, and other attributes, you can use:

```bash
udevadm monitor --environment --udev
```

This command will print real-time information about device events, including the values of `ACTION`, `KERNEL`, `SUBSYSTEM`, etc.

#### Example

When you plug in a USB device, the output might look like:

```bash
UDEV  [123456.789] add /devices/pci0000:00/0000:00:1d.0/usb1/1-1 (usb)
ACTION=add
DEVNAME=/dev/bus/usb/001/002
DEVPATH=/devices/pci0000:00/0000:00:1d.0/usb1/1-1
DEVTYPE=usb_device
ID_MODEL=Flash_Drive
ID_SERIAL=SanDisk_Cruzer
SUBSYSTEM=usb
```

### 5. Documentation and Man Pages

- **`man udev`**: Provides an overview of `udev` and its configuration.
- **`man udevadm`**: Details on the `udevadm` command, useful for managing and querying `udev`.

By combining these tools and methods, you can explore all the relevant information that can be used in your `udev` rules, allowing you to create precise and effective rules for managing devices on your system.
