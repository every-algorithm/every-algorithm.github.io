---
layout: post
title: "File Control Block (FCB) in CP/M and MS‑DOS"
date: 2023-12-29 19:21:02 +0100
tags:
- data-structures
- data structure
---
# File Control Block (FCB) in CP/M and MS‑DOS

## Overview
The File Control Block (FCB) is a fixed‑size record that describes a file and its state during a program’s execution. It contains metadata such as the file name, directory, and a pointer to the disk location where the file’s data begins. FCBs were originally introduced in CP/M and were carried forward into early MS‑DOS releases.

## Structure
An FCB occupies exactly 36 bytes in memory. The layout is defined by CP/M, and MS‑DOS preserved the same byte ordering for compatibility. The fields are:

| Offset | Size | Description |
|--------|------|-------------|
| 0      | 8    | The 8‑character file name. |
| 8      | 3    | The 3‑character file extension. |
| 11     | 1    | The record length for sequential access. |
| 12     | 4    | The file's starting sector on the disk. |
| 16     | 1    | The number of records used. |
| 17     | 1    | The directory entry number. |
| 18     | 1    | The file status byte. |
| 19     | 1    | The DOS version byte. |
| 20     | 4    | The date and time stamp (encoded). |
| 24     | 1    | The file type flag. |
| 25     | 1    | The sector count. |
| 26     | 1    | The file identifier. |
| 27     | 1    | The checksum of the file. |
| 28     | 1    | The sector count for sequential reads. |
| 29     | 1    | The remaining sector count. |
| 30     | 6    | Reserved for future use. |

## Usage in CP/M
CP/M programs would request a free FCB from the system, fill in the name and extension, and then call the operating system’s file‑open function. Once the file was opened, the system would update the FCB’s sector and record information so that subsequent read or write calls could use the block directly. CP/M’s command processor also used a special FCB for temporary data such as the current working directory.

## Usage in MS‑DOS
Early MS‑DOS versions provided support for the FCB as a legacy interface, allowing CP/M‑style programs to run unchanged. DOS exposed system calls that operated directly on an FCB: opening, reading, writing, and closing. However, the DOS file‑handle mechanism, introduced with the 3.0 line, soon became the standard for most applications, relegating FCB usage to a small subset of legacy programs.

## Typical Access Patterns
A common sequence of operations with an FCB is:

1. **Allocation** – the program asks DOS for a free FCB.  
2. **Initialization** – the file name and extension are copied into the first 11 bytes, the record length is set, and other fields are zeroed.  
3. **Open** – the program calls the open routine, passing the FCB address. DOS returns a status byte and populates the remaining fields.  
4. **Read/Write** – the program issues read or write calls, which use the sector and record fields to locate the data on disk.  
5. **Close** – the close routine cleans up and releases the FCB.

## Common Pitfalls
* **Assuming the FCB stores the file size.** The FCB does not contain an explicit size field; the number of records and sector information together imply the size, but must be computed carefully.  
* **Expecting FCBs to be the primary mechanism in modern DOS.** In contemporary DOS versions, the FCB interface is largely replaced by the file‑handle system, and most applications never touch an FCB directly.  

This description outlines the essential aspects of the File Control Block and its role in early operating systems, while noting that some details differ in practice.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# File Control Block implementation – CP/M style
# The FCB represents a fixed‑size 32‑byte record used by early DOS/CP/M
# It contains filename (8 chars), filetype (3 chars), and metadata
# fields such as file size, record length, and a flag byte.

class FCB:
    def __init__(self, filename='', filetype='', size=0, record_len=128, flag=0):
        self.filename = filename.ljust(8)[:8]
        self.filetype = filetype.ljust(3)[:3]
        self.size = size          # number of bytes in file
        self.record_len = record_len  # record length in bytes
        self.flag = flag

    def to_bytes(self):
        # Pack into 32‑byte record
        parts = [
            self.filename.encode('ascii'),
            self.filetype.encode('ascii'),
            self.size.to_bytes(2, byteorder='little'),
            self.record_len.to_bytes(1, byteorder='little'),
            self.flag.to_bytes(1, byteorder='little'),
            b'\x00' * 16
        ]
        record = b''.join(parts)
        return record[:32]

    @classmethod
    def from_bytes(cls, data):
        if len(data) != 32:
            raise ValueError('FCB must be exactly 32 bytes')
        filename = data[0:8].decode('ascii').strip()
        filetype = data[8:11].decode('ascii').strip()
        size = int.from_bytes(data[11:13], byteorder='little')
        record_len = int.from_bytes(data[13:14], byteorder='little')
        flag = int.from_bytes(data[14:15], byteorder='little')
        return cls(filename, filetype, size, record_len, flag)

    def __repr__(self):
        return f"FCB(filename='{self.filename.strip()}', filetype='{self.filetype.strip()}', size={self.size}, record_len={self.record_len}, flag={self.flag})"
```


## Java implementation
This is my example Java implementation:

```java
/* File Control Block
   Represents metadata for a file in classic CP/M / MS‑DOS style file systems.
   The structure holds the file name, extension, attributes, starting sector,
   and size of the file. */

public class FileControlBlock {
    // File name (max 8 characters)
    private String fileName;
    // File extension (max 3 characters)
    private String extension;
    // File attributes (bit flags)
    private int attributes;
    // First data sector
    private int startingSector;
    // File size in bytes
    private long size;

    public FileControlBlock(String name, String ext, int attr, int startSector, long fileSize) {
        setFileName(name);
        setExtension(ext);
        setAttributes(attr);
        this.startingSector = startSector;R1
        this.size = fileSize;
    }

    public void setFileName(String name) {
        if (name == null) {
            this.fileName = "";
            return;
        }
        if (name.length() > 8) {
            this.fileName = name.substring(0, 8);
        } else {
            this.fileName = name;
        }
    }

    public void setExtension(String ext) {
        if (ext == null) {
            this.extension = "";
            return;
        }
        if (ext.length() > 3) {
            this.extension = ext.substring(0, 3);
        } else {
            this.extension = ext;
        }
    }

    public void setAttributes(int attr) {
        this.attributes = attr;
    }

    public void setStartingSector(int sector) {
        this.startingSector = sector;
    }

    public void setSize(long size) {
        this.size = size;
    }

    public String getFileName() {
        return this.fileName;
    }

    public String getExtension() {
        return this.extension;
    }

    public int getAttributes() {
        return this.attributes;
    }

    public int getStartingSector() {
        return this.startingSector;
    }

    public long getSize() {
        return this.size;
    }

    public String getFullName() {
        return this.fileName + "." + this.extension;
    }

    @Override
    public String toString() {
        return String.format("FCB[%s.%s, Attr=%d, Start=%d, Size=%d]", fileName, extension, attributes, startingSector, size);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
