# Preamble

All OTDRs (Optical Time Delay Reflectometers) on the market today either save their data in the SOR (Standard OTDR Record) format or offer the ability to export to that format. The current version, which is reportedly defined by SR-4731 (Issue 2), is owned by Ericsson, which doesn’t offer a license that allows readers to write compliant software. In fact, the current license explicitly blocks you from creating any “Derivative works”, which, when asked for clarification, an Ericsson employee stated includes Open-Source (and even Closed-Source) software.

Fortunately, we are based in the UK, we haven’t purchased a license to their IP and have never had any knowledge of the IP’s contents, which, thanks to a bit of law the UK adopted from the EU (specifically Directive 91/250/EEC, Article 6), means we are legally allowed to reverse-engineer the API of the standard for the purpose of integrating with systems that use it. 

Unfortunately, from the example SOR files we’ve seen, every OTDR vendor (including every visualizer and library implementation) has their own unique interpretation of various fields. It’s almost as though they, too, have reverse-engineered the standard to avoid licensing issues. 

Now, [Sidney Li](https://morethanfootnotes.blogspot.com/2015/07/the-otdr-optical-time-domain.html?lr=1718804210501) (with some input from others) has already done most of the heavy lifting on reverse-engineering the SOR file format, so we are not taking credit for that. But as we needed to implement a clean C# Library, we figured we’d formalize the current consensus into what we believe could be a suitable open standard.

Readers of this document are encouraged to purchase an appropriate license to [SR-4731 (Issue 2) from Ericsson (formerly Telcordia)](https://telecom-info.njdepot.ericsson.net/site-cgi/ido/docs.cgi?ID=SEARCH&DOCUMENT=SR-4731&#ORD) if they desire to ensure that they have an implementation that is fully compliant with that standard.

What follows is a new standard that supports our independent version of an OTDR data storage format that may (or may not) be compatible with other OTDR data standards.

# Disclaimer

This document is written without knowledge of Ericsson’s (or any other rights holder’s) intellectual property and has been constructed independently by the Authors. Any compatibility with the Standard OTDR Record as defined by SR-4731 (Issue 2) or any derivative or preceding works thereof is circumstantial and is not guaranteed by the authors. 

Using this document with the intent of being compliant with any formats other than the format defined here is at your own risk.

If readers desire documentation that is fully compatible with the SOR format as defined by SR-4731 (or derivatives), then they should purchase the relevant documents from the rights holders of that standard.

# Copyright Statement
Copyright 2024 BaldrAI Ltd.

# OpenSOD - RFC 0001

This document defines the Open Standard OTDR Data (or OpenSOD for short). Any compatibility or lack thereof with other formats is circumstantial. 

To properly understand this document, some rudimentary knowledge of Optical Time Delay Reflectometry is recommended.

## File Extensions

While implementers are welcome to use (and support) any file extension they see fit, it is suggested that the`.SOD` extension is used.

## Definitions

`uint16` = A 16-bit long unsigned integer (a.k.a. an `unsigned short`).

`uint32` = A 32-bit long unsigned integer (a.k.a an `unsigned int`).

`int16` = A 16-bit long signed integer (a.k.a a `short`).

`int32` = A 32-bit long signed integer (a.k.a an `int`).

`char[n]`  = A fixed-length string of length `n`.

`string` = A NULL (`/0`) terminated string.

`list<T>` = A list of objects of type `T`.

## Top-level Data Structures

The file contains the following root-level data blocks:

- **Map** (REQUIRED & MUST BE FIRST) - Defines the name and length of all proceeding blocks
- **General Parameters** (REQUIRED) - Contains user-supplied information (e.g. cable id, fibre type, operator, etc.)
- **Supplier Parameters** (REQUIRED) - Contains device vendor information (e.g. supplier name, device model, software version etc.)
- **Fixed Parameters** (REQUIRED) - Contains configuration data (e.g. pulse width, index of refraction, wavelength, etc.)
- **Key Events** (REQUIRED; IF Data Points structure is missing) - Contains information about identified features (e.g. splices, couplings, bends, etc.)
- **Location Parameters** (OPTIONAL - OpenSOD Specific) - Contains information about expected features (e.g. longitude, latitude, sheath markers, etc.)
- **Data Points** (REQUIRED; IF Key Events structure is missing) - Contains raw measurement data as an array of data points
- **Proprietary Data** (OPTIONAL) - Contains vendor-specific data blobs
- **OpenSOD Footer** (REQUIRED & MUST BE LAST; IF using Location Parameters)

The general structure of the file is as follows:

```

                     1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3
|0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1|
|---/---/---/---/---/---/---/---/---/---/---/---/---/---/---/---|
|                                                               /
/                             Map                               /
/                                                               |
|---/---/---/---/---/---/---/---/---/---/---/---/---/---/---/---|
/                                                               /
/                          Body Data                            /
/                                                               /
/                             ...                               /
/                                                               |
|-+-+-+-+-+-+-+-+-+-+-|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
|        Footer       |
|-+-+-+-+-+-+-+-+-+-+-|
```

The file starts with the map, which is then followed by the other structured data blocks in the order defined in the map.

## Map

`BlkName = "MAP"` 

```
                     1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3
|0 1 2 3|4 5|6 7 8 9|0 1|2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
|-+-+-+-|-+-|-+-+-+-|-+-|---/---/---/---/---/---/---/---/---/---|
|BlkName|Ver|Length |Num|                                       /
|---/---/---/---/---/---/---/---/---/---/---/---/---/---/---/---|
/                          Map Data                             /
/                                                               |
|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
```

The Map header is similar in function and structure to a File Address Table (FAT) in the FAT32 filesystem. It provides the data needed to identify the location and size of each block within the file.

The `BLKName` field is a `string` that contains the word “`MAP`”. 

The `Ver` field is a `uint16` containing the version number of the file format - Files compatible with SR-4731 (Issue 2) typically use `2` here. OpenSOD files can use `1000` + the RFC number (e.g. for **OpenSOD RFC 0001**, the `Ver` would be `1001`). Files using OpenSOD versioning might not function with software and systems that don’t explicitly support OpenSOD.

The `Length` field is a `uint32` that indicates the end of the Header.

The `Num` field is a `uint16` that indicates the number of blocks defined in the `Map Data`

### Map Data

The `Map Data` section is a `list<MapData>` of `Num` items long.

The `MapData` structure is as follows:

```
           N N N N N N
           + + + + + +
|0-/---/-N|1 2|3 4 5 6|
|--/---/--|-+-|-+-+-+-|
|Name     |Ver|Length |
|-+-+-+-+-+-+-+-+-+-+-|
```

The `Name` is a `string` indicating the name of the block, expected values are provided in each block definition that follows.

The `Ver` field is a `uint16` containing the block’s version number, suggested values are provided in each block definition that follows.

The `Length` field is a `uint32` that indicates the size of the block.

## General Parameters

`Name = GenParams`

`Ver = 2`

The General Parameters block contains all the information about the fibre under test.

```
|--/---/--|-+-|--/---/--|--/---/--|-+-|-+-|--/---/--|--/---/--|--/---/--|
|Name     |Lng|CableId  |FibreId  |Typ|WaL|LocationA|LocationB|CableCode|
|-+-|-+-+-+-|-+-+-+-|--/---/--|--/---/--|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
|BuC|Offset |OffDist|Operator |Comments |
|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
```

The `Name` field is a `string` that matches the block name `GenParams`.

The `Lng` field is a `char[2]` that contains an ISO 639 Set 1 Country Code.

The `CableId` field is a `string` set by the user. It is normally used to indicate the patch panel ID at the test location.

The `FibreId` field is a `string` set by the user. It is normally used to indicate the patch panel port number at the test location.

The `Typ` field is a `uint16` containing the ITU-T G.xxx classification of the fibre under test (e.g. 657 for ITU-T G.657).

The `WaL` field is a `uint16` containing the nominal wavelength of the light pulse used in the test (e.g. 1310nm).

The `LocationA` field is a `string` set by the user. Most technicians leave this as the default (or blank), but some use it for the coordinates or the address of the A-side.

The `LocationB` field is a `string` set by the user. Most technicians leave this as the default (or blank), but some use it for the coordinates or the address of the B-side.

The `CableCode` field is a `string` whose use varies per vendor; some have it as a user-settable string, whereas others have it as the plaintext ITU-T G.xxx code of the fibre (e.g. ITU-T G.657).

The `BC` field is a `char[2]` that contains the Built Condition. We believe the common codes used here are:

`BC`: as-built / Build Condition,
`CC`: as-current / Current Condition / Pre-Work,
`RC`: as-repaired / Repaired Condition / Post-Work,
`NC`: as-new / New Condition / Post-Work,
`OT`: other.

The `Offset` field is a `uint32`, defining the length of the launch lead fibre measured in the units defined in the fixed parameters block as a fixed-point decimal with an exponent of 10^1.

The `OffDist` field is a `uint32`, the same as above, but as an integer dropping any decimal values.

The `Operator` field is a `string` set by the user. It is intended to be used for the Technician’s Name and/or Employee ID - This is normally configured once during device allocation.

The `Comments` field is a `string` set by the user. It is normally blank but is intended for the technician to write their notes in.

## Supplier Parameters

`Name = SupParams`

`Ver = 2`

The Supplier Parameters block contains information about the test device.

```
|--/---/--|--/---/--|--/---/--|--/---/--|--/---/--|--/---/--|--/---/--|--/---/--|
|Name     |SupplName|OTDRName |ODTRSn   |ModName  |ModuleSn |SoftwareV|Other    |
|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
```

The `Name` field is a `string` that matches the block name `SupParams`.

The `SupplName` field is a `string` that contains the vendor’s name.

The `OTDRName` field is a `string` that contains the device model number.

The `OTDRSn` field is a `string` that contains the device’s serial number

The `ModName` field is a `string` that contains the add-in module’s model number.

The `ModuleSn` field is a `string` that contains the add-in module’s serial number.

The `SoftwareV` field is a `string` that contains the software version number that was used when the file was written.

The `Other` field is a `string` that contains any other information the device’s vendor decided to include (frequently the calibration date).

## Fixed Parameters

`Name = FxdParams`

`Ver = 2`

The Fixed Parameters block contains the configuration used for the specific test.

```
                    |1 1 1 1|1 1|1 1|1 1 2 2|2 2 2 2|2 2|2 2 3 3
 0 1 2 3 4 5 6 7 8 9|0 1 2 3|4 5|6 7|8 9 0 1|2 3 4 5|6 7|8 9 0 1
|--/-------------/--|-+-+-+-|-+-|-+-|-+-+-+-|-+-+-+-|-+-|--/--/-|
|Name               |TimeStp|Unt|WaL|AcOffs |AcOffsD|NTr|       /
|---/---/---/---/---/---/---/---/---/---/---/---/---/---/---/---|
/                            PulseWidth                         /
|---/---/---/---/---/---/---/---/---/---/---/---/---/---/---/---|
/                          SampleSpacing                        /
|---/---/---/---/---/---/---/---/---/---/---/---/---/---/---/---|
/                    NumberofDataPointsInTrace                  /
|-+-+-+-|-+-|-+-+-+-|-+-|-+-+-+-|-+-+-+-|-+-+-+-|-+-|-+-|-+-|-+-|
|IoR    |BsC|NumAvgs|AvT|AcRange|AcRangD|FntPnlO|NFL|NFS|POf|LoT|
|-+-|-+-|-+-|-+-+-+-|-+-+-+-|-+-+-+-|-+-+-+-|-+-+-+-+-+-+-+-+-+-|
|ReT|EoT|TrT|X1     |Y1     | X2    |Y2     |
|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
```

The `Name` field is a `string` that matches the block name `FxdParams`.

The `TimeStp` field is a `uint32` that contains the time the test was done represented as the number of seconds since the Unix Epoch. This will overflow in 2038.

The `Unt` field is a `char[2]` that contains the Units used in Distance fields. The possible unit codes are:

`km`: Kilometers

`mt`: Meters (**DEFAULT**)

`ft`: Feet

`kf`: Thousand feet / Kilo-feet

`mi`: Miles

The `WaL` field is an `uint16`, which is the calibrated wavelength (in **nm**) used by the module for the test. This is supposed to be represented as a fixed-point decimal with an exponent of 10^1. This means this should be roughly 10x the wavelength detailed in the **General Parameters**.

The `AcOffs` field is an `int32` that defines the Acquisition Offset in nanoseconds as a fixed-point decimal with an exponent of 10^4 .  I.E., how much of the data was ignored due to launch blindness (the period of time during which the detector is blind after the pulse is sent/launched).

The `AcOffsD` field is an `int32`, this is a scaled version of the `AcOffs` field in the units defined in the `Unt` field as a fixed-point decimal with an exponent of 10^1.

The `NTr` field is a `uint16` that defines the number of traces present in the file. 

After the `NTr` field, the next three fields are arrays of length `NTr`. 

The `PulseWidth` field is a `list<uint16>` containing the pulse width used for each trace in the file.

The `SampleSpacing`field is a `list<uint32>` containing the sample spacing used for each trace in the file.

The `NumberOfDataPointsInTrace` field is a `list<uint32>` containing the number of data points in each trace.

The `IoR` field is a `uint32`, which is either detected or user-configured. It records the Index of Refraction of the fibre under test (either detected or user-configured) as a fixed-point decimal with an exponent of 10^5. To convert this back to its decimal format, use the following formula `float ior_f = ior_i / 100000.0`. If it’s unset, then a safe default IoR is around 1.46800 (the nominal IoR of G.652 @ 1550nm).

The `BsC` field is a `uint16`, which is either detected or user-configured. It records the Backscattering Coefficient of the fibre under test.

The `NumAvgs` field is a `uint32` that varies in use per vendor. In most cases, it represents the number of samples used in averages.

The `AvT` field is a `uint16` that stores the sample time used to calculate averages.

The `AcRange` field is a `uint32` that stores the acquisition range in **μs** as a fixed-point decimal with an exponent of 10^4. To convert this back to its decimal format, use the following formula `float acrange_f = acrange_i / 10000.0`.

The `AcRangD` field is a `uint32` that stores the acquisition range as a fixed-point decimal with an exponent of 10^1, scaled into the metric specified by the `Unt` field. This is done using the formula `float acrangd_f = acrange_f * (C / ior_f)`.

The `FntPnlO` field is an `int32` that stores the Front Panel Offset used by some implementations.

The `NFL` field is a `uint16` that stores the detected (or calibrated) noise floor, the lowest power level below which the 98th percentile of the signal noise is found.

The `NFS` field is a `uint16` that stores the detected (or calibrated) noise floor scaling factor.

The `POf` field is a `uint16` that stores the Power Offset / Attenuation used by some devices.

The `LoT` field is a `uint16` that stores the Loss Threshold (in dB as a fixed-point decimal with an exponent of 10^3) used to identify splice/non-reflective loss events to be stored in the Key Events block.

The `ReT` field is a `uint16` that stores the Reflection Threshold (in dB as a fixed-point decimal with an exponent of 10^3) used to identify coupling/fracture/reflective events to be stored in the Key Events block.

The `EoT` field is a `uint16` that stores the End of Transmission Threshold (in dB as a fixed-point decimal with an exponent of 10^3), which is used to identify the Launch and End of the Fibre events that will be stored in the Key Events block.

The `TrT` field is a `char[2]` that defines the type of trace. The codes used are:

`ST`: Standard/Forward Trace
`RT`: Reverse Trace
`DT`: Difference Trace
`RF`: Reference Trace

Following this are four `int32` fields, `X1`, `Y1`, `X2`, and `Y2`, that define a default viewing window used by some implementations.

## Key Events

`Name = KeyEvents` 

`Ver = 2`

```
Key Events
                     1|1 1|1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3
 0 1 2 3 4 5 6 7 8 9 0|1 2|3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
|-+-+-+-+-+-+-+-+-+-+-|-+-|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
|Name                 |NoE|                                     /
|---/---/---/---/---/---/---/---/---/---/---/---/---/---/---/---|
/                                                               /
/                         EventsArray                           /
/                                                               /
|---/---/---/---/---|-+-+-+-|-+-+-+-|-+-+-+-|-+-|-+-+-+-|-+-+-+-|
/                   |ETELo  |ETEL1  |ETEL2  |ORL|ORLM1  |ORLM2  |
|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
```

The `Name` field is a `string` that matches the block name `KeyEvents`.

The `NoE` field is a `uint16` that defines the number of `Events` that follow.

The `EventsArray` field is a `list<Event>` - defined below.

The `ETELo` field is an `int32` the represents the End-To-End Loss across the fibre, measured between the two points that follow.

The `ETEL1` field is an `uint32` the defines the position of the first marker used for the preceding measurement.

The `ETEL2` field is an `uint32` the defines the position of the second marker used for the preceding measurement.

The `ORL` field is an `int32` the represents the Optical Return Loss across the fibre, measured between the two points that follow.

The `ORLM1` field is an `uint32` the defines the position of the first marker used for the preceding measurement.

The `ORLM2` field is an `uint32` the defines the position of the second marker used for the preceding measurement.

### Event

The `Event` structure used in the `EventsArray` is formatted as follows:

```
Event
                    |1 1 1 1|1|1|1 1 1 1|2 2|2 2 2 2|2 2 2 2|3 3
 0 1|2 3 4 5|6 7|8 9|0 1 2 3|4|5|6 7 8 9|0 1|2 3 4 5|6 7 8 9|0 1
|-+-|-+-+-+-|-+-|-+-|-+-+-+-|-|-|-+-+-+-|-+-|-+-+-+-|-+-+-+-|-+-|
|EvN|Dist   |Slo|Los|Reflect|R|E|LndMkNo|Tec|Loc1   |Loc2   |Loc/
|-+-|-+-+-+-|-+-+-+-|--/---/--|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
/3  |Loc4   |Loc5   |Comment  /
|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
```

The `EvN` field is a `uint16` that specifies the `Event` number - Some vendors maintain event numbering and sorting, while others treat this as an unsorted array where the numbers don’t have any significant meaning and might not even be contiguous. *It is preferred that vendors treat this as a contiguous sequential index.*

The `Dist` field is a `uint32` that specifies the distance to the event in **μs** as a fixed-point decimal with an exponent of 10^4.

The `Slo` field is a `uint16` that contains the slope of the lead-in (the period between the previous event and this one),  as a fixed-point decimal with an exponent of 10^3. This is specified in dB/unit; using the units specified in the `FxdParams`.

The `Los` field is a `uint16` representing the loss caused by the event specified in dB as a fixed-point decimal with an exponent of 10^3.

The `Reflect` field is a `uint32` representing the peak of the reflection blindness relative to the current position in dBs of negative loss.

The `R` field is a `char[1]` that contains the Reflection Type number as an ASCII character. The values used are:

`0`: Non-reflective

`1`: Reflective (probably a fracture)

`2`: Saturated reflective (usually a start/end of fibre, sometimes a PC-type coupling)

The `E` field is a `char[1]` that contains the Event Type value as an ASCII character. The values used are:

`A`: Added by the user

`M`: Modified by the user

`E`: End of Fibre

`F`: Found by software

`O`: Out-of-Range

`D`: Modified end of fibre

The `LndMkNo` field is a `char[4]` containing the Landmark ID (set to `9999` if no landmarks are defined).

The `Tec` field is a `char[2]` containing the loss measurement technique. Common values used are:

`LS`: Least-Square

`2P`: Two-Point

The following five `Loc<n>` fields are `uint32` values and are sometimes used to store the following fields:

1. Distance from start to End of Previous Event / Slope Start
2. Distance from start to Beginning of Current Event / Slope End
3. Distance from start to End of Current Event
4. Distance from start to Beginning of the Next Event
5. Distance to the Peak of the Reflection.

The `Comment` field is a `string` that contains any user-specified notes about the event.

## Location Parameters

`Name = LocParams`

`Ver = 1001`

```
Location Parameters
                     1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3
 0 1 2 3 4 5 6 7|8 9|0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
|-+-+-+-+-+-+-+-|-+-|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
|Name           |NoL|                                           /
|---/---/---/---/---/---/---/---/---/---/---/---/---/---/---/---|
/                                                               /
/                        LocationsArray                         /
/                                                               /
|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
```

The `Name` field is a `string` that matches the block name `LocParams`.

The `NoL` field is a `uint16` that defines the number of `Locations` that follow.

The `LocationsArray` field is a `list<Location>`.

### Location

The `Location` structure is as follows:

```
Location
                    |1 1|1 1 1 1|1 1 1 1|2 2 2 2 2 -
|0 1 2 3|4 5 6 7|8 9|0 1|2 3 4 5|6 7 8 9|0 1 2 3 4 -
|-+-+-+-|-+-+-+-|-+-|-+-|-+-+-+-|-+-+-+-|-+-+-+-|--/---/--|
|LocNum |LocDist|Dif|EvN|Lat    |Long   |IdeLeLo|CableIn  |
|--/---/--|--/---/--|--/---/--|--/---/--|-+-+-+-+-+-+-+-+-|
|FibreIn  |CableOut |FibreOut |Comment  |
|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
```

The `LockNum` field is a `char[4]` containing the Location ID - If there is a matching `Event` this is the same value.

The `LocDist` field is a `uint32` containing the estimated distance in **μs** as a fixed-point decimal with an exponent of 10^4.

The `Diff` field is an `int16` containing the difference/drift between the `LocDist` value and the detected distance to the linked `Event`.

The `Evn` field is a `uint16` containing the linked `Event` number. If there is no matching event, it is set to 65535.

The `Lat` field is an `int32` defining the GPS Latitude as a fixed-point decimal with the exponent 10^7.

The `Long` field is an `int32` defining the GPS Longitude as a fixed-point decimal with the exponent 10^7.

The `IdeLeLo` field is a `uint32` that contains the Theoretical/Ideal Lead-in Loss (in dB) from the start of the fibre to the location as a fixed-point decimal with an exponent of 10^3.

The `CableIn` field is a `string` set by the user. It is used to indicate the outer sheath code of the fibre entering the location.

The `FibreIn` field is a `string` set by the user. It is used to indicate the inner sheath code (or colour) of the fibre (e.g. Yellow) entering the location.

The `CableOut` field is a `string` set by the user. It is used to indicate the outer sheath code of the fibre exiting the location.

The `FibreOut` field is a `string` set by the user. It is used to indicate the inner sheath code (or colour) of the fibre (e.g. Blue) exiting the location.

The `Comment` field is a `string`set by the user.

## Data Points

`Name = DataPts`

`Ver = 2`

```
Data Points
                     1 1|1 1|1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3
 0 1 2 3 4 5 6 7|8 9 0 1|2 3|4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
|-+-+-+-+-+-+-+-|-+-+-+-|-+-|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
|Name           |NumDPts|NTr|                                   /
|---/---/---/---/---/---/---/---/---/---/---/---/---/---/---/---|
/                                                               /
/                       TraceArray                              /
/                                                               /
|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
```

The `Name` field is a `string` that matches the block name `KeyEvents`.

The `NumDPts` field is a `uint32` defining the total number of data points/samples.

The `NTr` field is a `uint16` that defines the number of traces present.

### Traces

The `TraceArray` field is a `List<Trace>` 

```
Trace Array
                     1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3
 0 1 2 3|4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
|-+-+-+-|-+-|---/---/---/---/---/---/---/---/---/---/---/---/---|
|TrNDPts|ScF|                    DataPoints                     /
|-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
```

The `TrNDPts` field is a `uint32` defining the number of data points/samples.

The `ScF` field is a `uint16` that defines the scaling factor used to convert the following data points into dB of loss.

The `DataPoints` field is a `list<uint16>` of data points (of length `trNDPts`).

## OpenSOD Footer

`Name = Foot` 

`Ver = 1001`

```

                     1 
 0 1 2 3 4|5 6|7 8 9 0 
|-+-+-+-+-|-+-|-+-+-+-|
|Name     |Ver|OPENSOD|
|-+-+-+-+-+-+-+-+-+-+-+
```

The `Name` field is a `string` that matches the block name `KeyEvents`.

The `Ver` field is a `uint16` this value should be `1000` + the supported RFC number (e.g. for **OpenSOD RFC 0001**, the `Ver` would be `1001`).

The `OPENSOD` field is a `char[7]` that contains the word `"OPENSOD"`.
