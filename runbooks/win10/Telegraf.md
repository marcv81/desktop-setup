# Telegraf

## Installation

Download `https://dl.influxdata.com/telegraf/releases/telegraf-1.17.2_windows_amd64.zip`. Copy the binary to `C:\Program Files\telegraf`.

## Dependencies

### Open Hardware Monitor

Download OpenHardwareMonitor 0.9.5 from https://openhardwaremonitor.org/. Extract it. Copy all the files to `C:\Program Files\OpenHardwareMonitor`.

Download NSSM 2.24 from https://nssm.cc/download. Extract it. Copy the 64-bits binary to `C:\Program Files\nssm\`.

Start PowerShell as an administrator. Run the following.

    cd 'C:\Program Files\nssm\'
    .\nssm install OpenHardwareMonitor 'C:\Program Files\OpenHardwareMonitor\OpenHardwareMonitor.exe'

Open Services. Start OpenHardwareMonitor.

## Configuration

Create `C:\Program Files\telegraf\OpenHardwareMonitor.ps1`.

    $objects = get-wmiobject -namespace root\OpenHardwareMonitor -query 'select * from Sensor'
    Foreach ($o in $objects) {
      $name = $o.Name.Replace(" ", "\ ")
      Write-Host "openhardwaremonitor,parent=$($o.Parent),type=$($o.SensorType),index=$($o.Index),name=$name value=$($o.value)"
    }

Use the `C:\Program Files\telegraf\telegraf.conf` from the files directory.

Start PowerShell as an administrator. Run the following.

    cd 'C:\Program Files\telegraf\'
    .\telegraf --service install

Open Services. Start Telegraf.
