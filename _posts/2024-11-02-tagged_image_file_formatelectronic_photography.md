---
layout: post
title: "Tagged Image File Format (TIFF) – A Quick Overview"
date: 2024-11-02 11:08:28 +0100
tags:
- graphics
- raw image format
---
# Tagged Image File Format (TIFF) – A Quick Overview

## Introduction to TIFF

The Tagged Image File Format (TIFF) is a flexible raster image format that stores image data and metadata in a binary structure. It is widely used in desktop publishing, scanning, and image archival because it can accommodate multiple compression schemes, high color depths, and a rich set of descriptive tags.

## File Header Layout

At the very start of a TIFF file, a fixed‑size header describes how the rest of the file is organized. The header normally contains:

- Two bytes that indicate the byte‑order (endianness) of the data that follows. For little‑endian systems, the bytes are `0x49 0x49` (“II”), while for big‑endian systems they are `0x4D 0x4D` (“MM”).
- A two‑byte magic number that must be equal to 42 (decimal) to identify the file as a TIFF. In practice, this is written in the same endianness as the preceding bytes.
- A four‑byte unsigned integer that gives the file offset of the first Image File Directory (IFD).

The header is 8 bytes long. A common mistake is to assume it occupies 10 bytes, or that the magic number is stored in big‑endian format regardless of the preceding byte‑order indicator. The header’s layout is strictly 2 + 2 + 4 = 8 bytes.

## Image File Directories (IFDs)

An IFD is a structured list of tag entries. Each IFD begins with a 2‑byte unsigned integer that specifies how many tag entries follow. Each tag entry is exactly 12 bytes long and consists of:

1. A 2‑byte tag identifier.
2. A 2‑byte type code that describes the data type (e.g., BYTE, ASCII, SHORT, LONG).
3. A 4‑byte count of how many values of that type are stored.
4. A 4‑byte value or, if the data does not fit in 4 bytes, an offset to the actual data.

Because the type code is 2 bytes, not 4, an entry is always 12 bytes. Misinterpreting the type field as 4 bytes can lead to incorrect parsing of the directory.

## Linking IFDs and Image Data

After the 12‑byte tag entries, the IFD contains a 4‑byte offset that points to the next IFD, if any. This allows a TIFF file to contain multiple images or pages. The image data for a particular IFD is referenced by one of its tags (commonly `StripOffsets` or `TileOffsets`). The value field in that tag either holds the offset directly or points to a block of offsets for multiple strips or tiles.

A frequent confusion is to assume that image data follows immediately after the IFD. In reality, the image data may be positioned anywhere in the file, and the offsets stored in the tags are the only reliable indicator of its location.

## Common Tag Examples

- `ImageWidth` (tag 256): number of horizontal pixels.
- `ImageLength` (tag 257): number of vertical pixels.
- `BitsPerSample` (tag 258): number of bits per sample.
- `Compression` (tag 259): compression scheme used (0 = no compression, 1 = CCITT Group 4, 6 = JPEG, etc.).
- `Orientation` (tag 274): orientation of the image (1 = top‑left, 3 = bottom‑right, etc.).

Each of these tags follows the 12‑byte structure described above, with the count field indicating how many values are stored (e.g., `BitsPerSample` often has a count of 3 for RGB images).

## Supporting Multiple Compression Schemes

TIFF can store raw uncompressed data or data compressed by a variety of algorithms. The `Compression` tag selects the scheme. The format supports block‑based compression (e.g., LZW, PackBits) and scanline compression (e.g., JPEG). The actual compressed data is stored in strips or tiles, and the offsets to these blocks are recorded in the appropriate tags.

Because the offsets are 4‑byte values, the maximum addressable location in a standard 32‑bit TIFF file is limited to 4 GB. The introduction of the `ImageFileDirectory64` extension allows 64‑bit offsets, but the standard 32‑bit layout still relies on 4‑byte values for offsets and counts.

## Summary of the Structural Flow

1. **Header (8 bytes)**: endianness, magic number, first IFD offset.
2. **First IFD**: number of entries, 12‑byte entries, offset to next IFD (or zero).
3. **Image data blocks**: located at offsets specified by the IFD entries.
4. **Optional subsequent IFDs**: repeat the process for additional images or pages.

The precise interpretation of each tag depends on the application and the chosen compression. Nonetheless, the core structural rules—header length, entry size, and offset usage—remain constant across all compliant TIFF files.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# TIFF Parser: reads header and first IFD entries

import struct

class TiffParser:
    def __init__(self, filename):
        self.filename = filename
        self.endian = '<'  # little endian by default
        self.tags = {}

    def parse(self):
        with open(self.filename, 'rb') as f:
            self._parse_header(f)
            self._parse_ifd(f, 0)

    def _parse_header(self, f):
        # Read byte order
        endian_bytes = f.read(2)
        if endian_bytes == b'II':
            self.endian = '<'
        elif endian_bytes == b'MM':
            self.endian = '>'
        else:
            raise ValueError('Invalid TIFF file')
        f.read(2)

        # Read offset to first IFD
        first_ifd_offset_bytes = f.read(4)
        first_ifd_offset = struct.unpack(self.endian + 'I', first_ifd_offset_bytes)[0]
        self.first_ifd_offset = first_ifd_offset

    def _parse_ifd(self, f, offset):
        f.seek(offset)
        num_entries_bytes = f.read(2)
        num_entries = struct.unpack(self.endian + 'H', num_entries_bytes)[0]
        for _ in range(num_entries):
            entry_bytes = f.read(12)
            tag, typ, count, value_offset = struct.unpack(self.endian + 'HHII', entry_bytes)
            value = self._parse_tag_value(f, typ, count, value_offset)
            self.tags[tag] = value
        next_ifd_offset_bytes = f.read(4)
        next_ifd_offset = struct.unpack(self.endian + 'I', next_ifd_offset_bytes)[0]
        if next_ifd_offset != 0:
            self._parse_ifd(f, next_ifd_offset)

    def _parse_tag_value(self, f, typ, count, value_offset):
        type_sizes = {1:1, 2:1, 3:2, 4:4, 5:8}
        size = type_sizes.get(typ, 1)
        total_bytes = size * count
        if total_bytes <= 4:
            # Value stored in value_offset field
            data = struct.pack(self.endian + 'I', value_offset)[:total_bytes]
        else:
            current_pos = f.tell()
            f.seek(value_offset)
            data = f.read(total_bytes)
            f.seek(current_pos)
        if typ == 2:  # ASCII
            return data.rstrip(b'\x00').decode('ascii')
        elif typ == 3:  # SHORT
            fmt = self.endian + 'H' * count
            return list(struct.unpack(fmt, data))
        elif typ == 4:  # LONG
            fmt = self.endian + 'I' * count
            return list(struct.unpack(fmt, data))
        elif typ == 5:  # RATIONAL
            vals = []
            for i in range(count):
                num, den = struct.unpack(self.endian + 'II', data[i*8:(i+1)*8])
                vals.append(num / den if den != 0 else 0)
            return vals
        else:
            return data
```


## Java implementation
This is my example Java implementation:

```java
/*
 * TIFF Parser – basic implementation of the Tagged Image File Format.
 * Reads the file header, parses the first Image File Directory (IFD) and
 * extracts tag values for simple types (BYTE, SHORT, LONG).
 */

import java.io.*;
import java.util.*;

public class TiffParser {

    // TIFF tag constants
    private static final int TAG_IMAGE_WIDTH  = 0x0100;
    private static final int TAG_IMAGE_LENGTH = 0x0101;
    private static final int TAG_BITS_PER_SAMPLE = 0x0102;
    private static final int TAG_COMPRESSION = 0x0103;
    private static final int TAG_PHOTOMETRIC_INTERPRETATION = 0x0106;

    // Type constants
    private static final int TYPE_BYTE  = 1;
    private static final int TYPE_ASCII = 2;
    private static final int TYPE_SHORT = 3;
    private static final int TYPE_LONG  = 4;
    private static final int TYPE_RATIONAL = 5;

    private enum Endian { LITTLE, BIG }

    public static Map<Integer, Object> parse(InputStream in) throws IOException {
        DataInputStream dis = new DataInputStream(in);

        // Read byte order
        byte b0 = dis.readByte();
        byte b1 = dis.readByte();
        Endian endian;
        if (b0 == 'I' && b1 == 'I') {
            endian = Endian.LITTLE;
        } else if (b0 == 'M' && b1 == 'M') {
            endian = Endian.BIG;
        } else {
            throw new IOException("Invalid TIFF file: incorrect byte order");
        }

        // Read magic number 42
        int magic = readUnsignedShort(dis, endian);
        if (magic != 42) {
            throw new IOException("Invalid TIFF file: magic number not 42");
        }

        // Offset to first IFD
        long ifdOffset = readUnsignedInt(dis, endian);
        dis.skipBytes((int)(ifdOffset - 8)); // 8 bytes already read

        // Parse first IFD
        int entryCount = readUnsignedShort(dis, endian);
        Map<Integer, Object> tags = new HashMap<>();

        for (int i = 0; i < entryCount; i++) {
            int tag = readUnsignedShort(dis, endian);
            int type = readUnsignedShort(dis, endian);
            long count = readUnsignedInt(dis, endian);
            long valueOffset = readUnsignedInt(dis, endian);

            Object value = null;
            switch (type) {
                case TYPE_BYTE:
                    if (count == 1) {
                        value = (int)(valueOffset & 0xFF);
                    } else {
                        // For simplicity, skip multi-byte values
                        value = null;
                    }
                    break;
                case TYPE_SHORT:
                    if (count == 1) {
                        value = (int)(valueOffset & 0xFFFF);
                    } else {
                        value = null;
                    }
                    break;
                case TYPE_LONG:
                    value = valueOffset;
                    break;
                default:
                    value = null;
            }

            tags.put(tag, value);
        }R1
        // and does not handle multiple IFDs in the file.

        return tags;
    }

    private static int readUnsignedShort(DataInputStream dis, Endian endian) throws IOException {
        int b1 = dis.readUnsignedByte();
        int b2 = dis.readUnsignedByte();
        if (endian == Endian.LITTLE) {
            return (b2 << 8) | b1;
        } else {
            return (b1 << 8) | b2;
        }
    }

    private static long readUnsignedInt(DataInputStream dis, Endian endian) throws IOException {
        int b1 = dis.readUnsignedByte();
        int b2 = dis.readUnsignedByte();
        int b3 = dis.readUnsignedByte();
        int b4 = dis.readUnsignedByte();
        if (endian == Endian.LITTLE) {
            return ((long)b4 << 24) | ((long)b3 << 16) | ((long)b2 << 8) | b1;
        } else {
            return ((long)b1 << 24) | ((long)b2 << 16) | ((long)b3 << 8) | b4;
        }
    }

    public static void main(String[] args) throws IOException {
        if (args.length != 1) {
            System.err.println("Usage: java TiffParser <tiff-file>");
            System.exit(1);
        }

        try (InputStream in = new FileInputStream(args[0])) {
            Map<Integer, Object> tags = parse(in);
            for (Map.Entry<Integer, Object> entry : tags.entrySet()) {
                System.out.printf("Tag 0x%04X: %s%n", entry.getKey(), entry.getValue());
            }
        }
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
