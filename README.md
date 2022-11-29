# nvidia-data

The latest product family (GPU) and operating system data from the NVIDIA Download API ([lookupValueSearch](https://www.nvidia.com/Download/API/lookupValueSearch.aspx)) as JSON files.

## What does the script do?

```get_data.py``` reads XML data from the NVIDIA Download API and converts it to a form that can be read from more easily (key value pairs) when comparing to corresponding data from an OS.

![Data mapping](https://i.ibb.co/q9295fg/data-mapping.png "Data mapping")

Desktop and notebook GPUs are separated to account for some GPUs being present in both a desktop and notebook series (e.g. GeForce GTX 1050 Ti); some notebook GPUs may still be in the desktop key (e.g. T1000, ) because NVIDIA listed them under a desktop series, both a desktop and notebook series or none at all. OS data is structured more traditionally, since both the code and part of the name must be compared.

GPU data example (GPU name and `pfid`):

```json
{
    "GeForce RTX 3070": "933",
}
```

OS data example (OS code, OS name and `osID`):

```json
[
    {
        "code": "10.0",
        "name": "Windows 10 64-bit",
        "id": "57"
    },
]
```

## Notes

- Product family (GPU) data ([gpu-data.json](https://raw.githubusercontent.com/ZenitH-AT/nvidia-data/main/gpu-data.json)) is updated whenever the results of ```https://www.nvidia.com/Download/API/lookupValueSearch.aspx?TypeID=3``` change
	- Updates to this file will always occur before any new GPUs are released
- Operating system data ([os-data.json](https://raw.githubusercontent.com/ZenitH-AT/nvidia-data/main/os-data.json)) is updated whenever the results of ```https://www.nvidia.com/Download/API/lookupValueSearch.aspx?TypeID=4``` change
- Since almost all NVIDIA products are GPUs, product family is referred to as GPU throughout this repository
- GPU data keys are run through a ```clean_gpu_name()``` function to aim to match the GPU name reported by the OS (without the "NVIDIA " prefix, " with Max-Q Design" suffix, etc.):

    Old name | New name
    --- | ---
    GeForce 7050 / NVIDIA nForce 610i | GeForce 7050
    nForce 610i/GeForce 7050     | nForce 610i
    Quadro M6000 24GB | Quadro M6000
    GeForce GTX 760 Ti (OEM) | GeForce GTX 760 Ti
    NVIDIA TITAN RTX | TITAN RTX
    GeForce GTX 1660 SUPER | GeForce GTX 1660 Super
    
    - Please open issues with any discrepancies you find

## Running the script

If the JSON files in this repository are outdated, they can be generated by running the following commands:

```
git clone https://github.com/ZenitH-AT/nvidia-data
cd nvidia-data
pip install -r requirements.txt
python get_data.py
```

## Where are the JSON files used?

The JSON files created by ```get_data.py``` were initially primarily intended to be used by the ```nvidia-update.ps1``` script (available at [ZenitH-AT/nvidia-update](https://github.com/ZenitH-AT/nvidia-update)) but are now also used by other projects, such as [ElPumpo/TinyNvidiaUpdateChecker](https://github.com/ElPumpo/TinyNvidiaUpdateChecker).

Previously, ```nvidia-update.ps1``` needed to query the NVIDIA Download API multiple times, iterating through and filtering every bit of data. Measures to speed up this process (i.e. limiting queries to desktop/mobile GPU and GeForce cards only) mostly just made the code more complicated.

Now, the script passes the data taken directly from this repository to the NVIDIA [AjaxDriverService](https://gfwsl.geforce.com/services_toolkit/services/com/nvidia/services/AjaxDriverService.php). An example of how this data is used can be found [here](https://github.com/ZenitH-AT/nvidia-update#faq).

## Possible future changes

- The script should be updated to account for not all alternative names being repeated in other entries (e.g. nForce 630i is not retrieved by ```get_data.py```).