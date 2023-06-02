#include <Uefi.h>
#include <Library/UefiLib.h>
#include <Library/UefiBootServicesTableLib.h>
#include <Protocol/UsbIo.h>

EFI_STATUS EFIAPI ShellAppMain(IN UINTN Argc, IN CHAR16 **Argv)
{
    EFI_STATUS Status;
    EFI_HANDLE *HandleBuffer;
    UINTN HandleCount;
    BOOLEAN UsbDevicesFound = FALSE;
    BOOLEAN BootSystem = TRUE;

    // Locate all USB I/O handles
    Status = gBS->LocateHandleBuffer(ByProtocol, &gEfiUsbIoProtocolGuid, NULL, &HandleCount, &HandleBuffer);
    if (!EFI_ERROR(Status))
    {
        UsbDevicesFound = TRUE;

        // Check if any USB device is a keyboard or mouse
        for (UINTN i = 0; i < HandleCount; i++)
        {
            EFI_USB_IO_PROTOCOL *UsbIo;
            Status = gBS->HandleProtocol(HandleBuffer[i], &gEfiUsbIoProtocolGuid, (VOID **)&UsbIo);
            if (!EFI_ERROR(Status))
            {
                EFI_USB_INTERFACE_DESCRIPTOR InterfaceDescriptor;
                Status = UsbIo->UsbGetInterfaceDescriptor(UsbIo, &InterfaceDescriptor);
                if (!EFI_ERROR(Status))
                {
                    if (InterfaceDescriptor.InterfaceClass == 0x03 && (InterfaceDescriptor.InterfaceProtocol == 0x01 || InterfaceDescriptor.InterfaceProtocol == 0x02))
                    {
                        // Keyboard or mouse found, boot system
                        BootSystem = TRUE;
                        break;
                    }
                }
            }
        }

        gBS->FreePool(HandleBuffer);
    }

    if (UsbDevicesFound)
    {
        Print(L"USB devices found\n");
    }
    else
    {
        Print(L"USB devices not found\n");
        BootSystem = TRUE;
    }

    if (BootSystem)
    {
        // Boot system
        EFI_INPUT_KEY Key;
        Print(L"Press any key to continue...\n");
        gBS->WaitForEvent(1, &gST->ConIn->WaitForKey, NULL);
        gST->ConIn->ReadKeyStroke(gST->ConIn, &Key);
        gBS->Stall(2000000); // 2-second delay
        gBS->ExitBootServices(gImageHandle, Key.ScanCode);
    }

    return EFI_SUCCESS;
}
