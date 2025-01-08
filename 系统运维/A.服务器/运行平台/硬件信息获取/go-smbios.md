项目地址：

- <https://github.com/siderolabs/go-smbios>

## SMBIOS 信息

可提供的信息

### （1）SMBIOS Version

<https://github.com/siderolabs/go-smbios/blob/v0.3.3/smbios/smbios.go#L18-L22>

SMBIOS 的版本，例如 3.2.0

```go
smbios.Version {
    Major: 3,
    Minor: 2,
    Revision: 0,
}

smbios.Version {
    Major: 3,
    Minor: 5,
    Revision: 0,
}
```

### （2）BIOSInformation

<https://github.com/siderolabs/go-smbios/blob/v0.3.3/smbios/bios_information.go#L10-L17>

BIOS information

- Vendor
- Version
- ReleaseDate

```go
smbios.BIOSInformation {
    Vendor: "American Megatrends Inc.",
    Version: "2.7",
    ReleaseDate: "09/21/2023",
}

smbios.BIOSInformation {
    Vendor: "American Megatrends Inc.",
    Version: "0403",
    ReleaseDate: "02/06/2023",
}
```

### （3）SystemInformation

<https://github.com/siderolabs/go-smbios/blob/v0.3.3/smbios/system_information.go#L20C6-L38>

SMBIOS system information

- Manufacturer
- ProductName
- Version
- SerialNumber
- UUID
- WakeUpType
- SKUNumber
- Family

```go
smbios.SystemInformation {
    Manufacturer: "Supermicro",
    ProductName: "AS -4124GS-TNR",
    Version: "0123456789",
    SerialNumber: "A404070X3B11344",
    UUID: "9b582200-7765-11ed-8000-7cc2554dfdda",
    WakeUpType: WakeUpTypePowerSwitch (6),
    SKUNumber: "",
    Family: "",
}

smbios.SystemInformation {
    Manufacturer: "ASUS",
    ProductName: "System Product Name",
    Version: "System Version",
    SerialNumber: "System Serial Number",
    UUID: "a01149a6-1158-ab22-8d48-581122ab8d47",
    WakeUpType: WakeUpTypePowerSwitch (6),
    SKUNumber: "SKU",
    Family: "",
}
```

### （4）BaseboardInformation

<https://github.com/siderolabs/go-smbios/blob/v0.3.3/smbios/baseboard_information.go#L9-L31>

SMBIOS baseboard information

- Manufacturer
- Product
- Version
- SerialNumber
- AssetTag
- LocationInChassis
- BoardType

```go
smbios.BaseboardInformation {
    Manufacturer: "Supermicro",
    Product: "H12DSG-O-CPU",
    Version: "1.01A",
    SerialNumber: "VM22CS600902",
    AssetTag: "",
    LocationInChassis: "",
    BoardType: BoardTypeProcessorMemoryModule (10),
}

smbios.BaseboardInformation {
    Manufacturer: "ASUSTeK COMPUTER INC.",
    Product: "Pro WS W790-ACE",
    Version: "Rev 1.xx",
    SerialNumber: "230215393300040",
    AssetTag: "Default string",
    LocationInChassis: "Default string",
    BoardType: BoardTypeProcessorMemoryModule (10),
}
```

### （5）SystemEnclosure

<https://github.com/siderolabs/go-smbios/blob/v0.3.3/smbios/system_enclosure.go#L9-L21>

system enclosure

```go
smbios.SystemEnclosure {
    Manufacturer: "Supermicro",
    Version: "0123456789",
    SerialNumber: "C4180AM45AC0293",
    AssetTagNumber: "",
    SKUNumber: "",
}

smbios.SystemEnclosure {
    Manufacturer: "Intel Corporation",
    Version: "0.1",
    SerialNumber: "Default string",
    AssetTagNumber: "",
    SKUNumber: "Default string",
}
```

### （6）ProcessorInformation

<https://github.com/siderolabs/go-smbios/blob/v0.3.3/smbios/processor_information.go#L9-L46>

```go
[]smbios.ProcessorInformation len: 2, cap: 2, [
    {
        SocketDesignation: "CPU1",
        ProcessorManufacturer: "Advanced Micro Devices, Inc.",
        ProcessorVersion: "AMD EPYC 7742 64-Core Processor",
        MaxSpeed: 3400,
        CurrentSpeed: 2250,
        Status: 65,
        SerialNumber: "Unknown",
        AssetTag: "Unknown",
        PartNumber: "Unknown",
        CoreCount: 64,
        CoreEnabled: 64,
        ThreadCount: 128
    },{
        SocketDesignation: "CPU2",
        ProcessorManufacturer: "Advanced Micro Devices, Inc.",
        ProcessorVersion: "AMD EPYC 7742 64-Core Processor",
        MaxSpeed: 3400,
        CurrentSpeed: 2250,
        Status: 65,
        SerialNumber: "Unknown",
        AssetTag: "Unknown",
        PartNumber: "Unknown",
        CoreCount: 64,
        CoreEnabled: 64,
        ThreadCount: 128,
    },
]

[]smbios.ProcessorInformation len: 1, cap: 1, [
    {
        SocketDesignation: "CPU0",
        ProcessorManufacturer: "Intel(R) Corporation",
        ProcessorVersion: "Intel(R) Xeon(R) w7-2475X",
        MaxSpeed: 4000,
        CurrentSpeed: 2600,
        Status: 65,
        SerialNumber: "",
        AssetTag: "UNKNOWN",
        PartNumber: "",
        CoreCount: 20,
        CoreEnabled: 20,
        ThreadCount: 40,
    },
]
```

### （7）MemoryDevice

<https://github.com/siderolabs/go-smbios/blob/v0.3.3/smbios/memory_device.go#L14-L134>

```bash
[]smbios.MemoryDevice len: 32, cap: 36, [
    {
        PhysicalMemoryArrayHandle: 53,
        MemoryErrorInformationHandle: 60,
        TotalWidth: 72,
        DataWidth: 64,
        Size: 32767,
        FormFactor: FormFactorTSOP (9),
        DeviceSet: "",
        DeviceLocator: "P1-DIMMA1",
        BankLocator: "P0_Node0_Channel0_Dimm0",
        MemoryType: MemoryTypeLPDDR3 (26),
        TypeDetail: 128,
        Speed: 3200,
        Manufacturer: "Samsung",
        SerialNumber: "H0KE0003094462559C",
        AssetTag: "P1-DIMMA1_AssetTag (date:23/09)",
        PartNumber: "M393A8G40BB4-CWE",
        Attributes: 2,
        ExtendedSize: 0,
        ConfiguredMemorySpeed: 2933,
        MinimumVoltage: 1200,
        MaximumVoltage: 1200,
        ConfiguredVoltage: 1200,
    },{
        PhysicalMemoryArrayHandle: 53,
        MemoryErrorInformationHandle: 63,
        TotalWidth: 72,
        DataWidth: 64,
        Size: 32767,
        FormFactor: FormFactorTSOP (9),
        DeviceSet: "",
        DeviceLocator: "P1-DIMMA2",
        BankLocator: "P0_Node0_Channel0_Dimm1",
        MemoryType: MemoryTypeLPDDR3 (26),
        TypeDetail: 128,
        Speed: 3200,
        Manufacturer: "Samsung",
        SerialNumber: "H0KE000309446233CE",
        AssetTag: "P1-DIMMA2_AssetTag (date:23/09)",
        PartNumber: "M393A8G40BB4-CWE",
        Attributes: 2,
        ExtendedSize: 0,
        ConfiguredMemorySpeed: 2933,
        MinimumVoltage: 1200,
        MaximumVoltage: 1200,
        ConfiguredVoltage: 1200,
    },{
        PhysicalMemoryArrayHandle: 53,
        MemoryErrorInformationHandle: 66,
        TotalWidth: 72,
        DataWidth: 64,
        Size: 32767,
        FormFactor: FormFactorTSOP (9),
        DeviceSet: "",
        DeviceLocator: "P1-DIMMB1",
        BankLocator: "P0_Node0_Channel1_Dimm0",
        MemoryType: MemoryTypeLPDDR3 (26),
        TypeDetail: 128,
        Speed: 3200,
        Manufacturer: "Samsung",
        SerialNumber: "H0KE000309446252D2",
        AssetTag: "P1-DIMMB1_AssetTag (date:23/09)",
        PartNumber: "M393A8G40BB4-CWE",
        Attributes: 2,
        ExtendedSize: 0,
        ConfiguredMemorySpeed: 2933,
        MinimumVoltage: 1200,
        MaximumVoltage: 1200,
        ConfiguredVoltage: 1200,
    },
]

[]smbios MemoryDevice len: 8, cap: 8, [
    {
        PhysicalMemoryArrayHandle: 20,
        MemoryErrorInformationHandle: 65534,
        TotalWidth: 80,
        DataWidth: 64,
        Size: 32767,
        FormFactor: FormFactorTSOP (9),
        DeviceSet: "",
        DeviceLocator: "CPU0_DIMM_A1",
        BankLocator: "NODE 0",
        MemoryType: MemoryTypeDRAM|MemoryTypeLPDDR5 (34),
        TypeDetail: 128,
        Speed: 4800,
        Manufacturer: "Samsung",
        SerialNumber: "029836ED",
        AssetTag: "CPU0_DIMM_A1_AssetTag",
        PartNumber: "M321R4GA3BB6-CQKDG",
        Attributes: 2,
        ExtendedSize: 32768,
        ConfiguredMemorySpeed: 4800,
        MinimumVoltage: 1100,
        MaximumVoltage: 1100,
        ConfiguredVoltage: 1100
    }, {
        PhysicalMemoryArrayHandle: 20,
        MemoryErrorInformationHandle: 65534,
        TotalWidth: 80,
        DataWidth: 64,
        Size: 32767,
        FormFactor: FormFactorTSOP (9),
        DeviceSet: "",
        DeviceLocator: "CPU0_DIMM_A2",
        BankLocator: "NODE 0",
        MemoryType: MemoryTypeDRAM|MemoryTypeLPDDR5 (34),
        TypeDetail: 128,
        Speed: 4800,
        Manufacturer: "Samsung",
        SerialNumber: "029832FC",
        AssetTag: "CPU0_DIMM_A2_AssetTag",
        PartNumber: "M321R4GA3BB6-CQKDG",
        Attributes: 2,
        ExtendedSize: 32768,
        ConfiguredMemorySpeed: 4800,
        MinimumVoltage: 1100,
        MaximumVoltage: 1100,
        ConfiguredVoltage: 1100
    }, {
        PhysicalMemoryArrayHandle: 20,
        MemoryErrorInformationHandle: 65534,
        TotalWidth: 80,
        DataWidth: 64,
        Size: 32767,
        FormFactor: FormFactorTSOP (9),
        DeviceSet: "",
        DeviceLocator: "CPU0_DIMM_B1",
        BankLocator: "NODE 0",
        MemoryType: MemoryTypeDRAM|MemoryTypeLPDDR5 (34),
        TypeDetail: 128,
        Speed: 4800,
        Manufacturer: "Samsung",
        SerialNumber: "029836EB",
        AssetTag: "CPU0_DIMM_B1_AssetTag",
        PartNumber: "M321R4GA3BB6-CQKDG",
        Attributes: 2,
        ExtendedSize: 32768,
        ConfiguredMemorySpeed: 4800,
        MinimumVoltage: 1100,
        MaximumVoltage: 1100,
        ConfiguredVoltage: 1100
    },
]
```

