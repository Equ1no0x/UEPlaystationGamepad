# Playstation Gamepad For Unreal Engine 5.0.3

<img src="https://cdn.discordapp.com/attachments/1185312433024282764/1212595159053639811/UEPlaystationGamepad.png?ex=65f2683e&is=65dff33e&hm=c17c1acf1280e9b9d4100925aaf68d7cf65ba8441d1eb7e222614f8746420cce&" width="640px" /> <br/>

Playstation Gamepad For Unreal Engine 5.0.3 using a modified Raw Input. This project is set up to enable DualShock3/Sixaxis, DualShock4, and DualSense controllers to bind with Unreal Engine Gamepad object, you can use your Playstation controllers for Gameplay, CommonUI, and EnhancedInput with the same Gamepad code like other XInput gamepads instead of fork to GenericUSBController.<br/>

## About

Originally to make DualShock 4 work with Unreal Engine 4 was just a list of RawInputWindows Plugin settings and was simpler to describe like in this forum post <a href="https://forums.unrealengine.com/t/tutorial-ue4-using-dualshock4-controller-via-usb-ps4-ds4-gamepad/133314" target="_blank">[Tutorial] UE4 using Dualshock4 controller (via USB, PS4 DS4 Gamepad)</a>.<br/>

## Notes

- You can have all 3 controllers connected to your PC USB.
	* DualShock3/Sixaxis controllers take priority	
- Basic Enhanced Input Action setup (only Buttons and Axes - Pressed and Released), for more information on how to expand and create more Input Actions check <a href="https://docs.unrealengine.com/5.1/en-US/enhanced-input-in-unreal-engine/" target="_blank">Enhanced Input - An overview of the Enhanced Input Plugin.</a> and <a href="https://dev.epicgames.com/community/learning/tutorials/eD13/unreal-engine-enhanced-input-in-ue5" target="_blank">Enhanced Input in UE5 - Official Tutorial</a>.<br/>

## DPad compatibility with CommonUI

The Playstation controllers DPad use Axis1D directional inputs, the Axis1D register from 0.0f to 8.0f : 0=Top, 2=Right, 4=Bottom, 6=Left, 8=No Input.<br/><br/>
Unreal Engine doesn't expect DPad directions from axis values, only as buttons pressed/released, to enable compatibility with CommonUI, EnhancedInput, and other XInput gamepads you need to modify the *RawInput Plugin* with the following changes:<br/><br/>
If you prefer you can download my source files from:<br/>
<a href="https://github.com/Equ1no0x/UEPlaystationGamepad/blob/5.0.3/.git_files/RawInputWindows.h" target="_blank">RawInputWindows.h</a><br/>
<a href="https://github.com/Equ1no0x/UEPlaystationGamepad/blob/5.0.3/.git_files/RawInputWindows.cpp" target="_blank">RawInputWindows.cpp</a><br/>
<br/>
*Engine\Plugins\Experimental\RawInput\Source\RawInput\Public\Windows\RawInputWindows.h*<br/>
*Line 322 at the end of the file, add* :<br/>
```c++
//Playstation DPad. DPadMap Index size for 8 inputs
FName DPadMap[7] = {};
struct PlaystationID {
	int32 vID; // VendorID
	int32 pID; // ProductID
	int32 aID; // Array Index
} psID[3];
```
<br/>

*Engine\Plugins\Experimental\RawInput\Source\RawInput\Private\Windows\RawInputWindows.cpp*<br/>
*Line 84, inside void FRawWindowsDeviceEntry::InitializeNameArrays(), add* : <br/>
```c++
	DPadMap[0] = FGamepadKeyNames::DPadUp;
	DPadMap[2] = FGamepadKeyNames::DPadRight;
	DPadMap[4] = FGamepadKeyNames::DPadDown;
	DPadMap[6] = FGamepadKeyNames::DPadLeft;

	psID[0] = { 1356, 1476, 4 }; // DS4 GEN1
	psID[1] = { 1356, 2508, 4 }; // DS4 GEN2
	psID[2] = { 1356, 3302, 7 }; // DualSense
```
<br/>

*Line 1049, inside if (DeviceEntry.bNeedsUpdate) { before the endif }, add* : <br/>
```c++
			for (PlaystationID& ps : psID) {
				if (DeviceEntry.DeviceData.VendorID == ps.vID && DeviceEntry.DeviceData.ProductID == ps.pID) {
					FAnalogData* DPadAxis1D = &DeviceEntry.AnalogData[ps.aID];
					int iPrevValue = FMath::FloorToInt(DPadAxis1D->PreviousValue);
					int iValue = FMath::FloorToInt(DPadAxis1D->Value);
					bool bIsRepeat = iValue == iPrevValue;

					if (!bIsRepeat) {
						if (iPrevValue != 8) {
							if (iPrevValue % 2 == 1) {
								MessageHandler->OnControllerButtonReleased(DPadMap[iPrevValue - 1], 0, bIsRepeat);
								iPrevValue++;
								if (iPrevValue == 8) iPrevValue = 0;
							}
							MessageHandler->OnControllerButtonReleased(DPadMap[iPrevValue], 0, bIsRepeat);
						}

						if (iValue != 8) {
							if (iValue % 2 == 1) {
								MessageHandler->OnControllerButtonPressed(DPadMap[iValue - 1], 0, bIsRepeat);
								iValue++;
								if (iValue == 8.0f) iValue = 0.0f;
							}
							MessageHandler->OnControllerButtonPressed(DPadMap[iValue], 0, bIsRepeat);
						}
						DPadAxis1D->PreviousValue = DPadAxis1D->Value;
					}
				}
			}
		}
	}
}
```

## Credits

Unreal Engine from Epic Games - https://www.unrealengine.com/ <br/>
Playstation DualShock4 Icons by ArksDigital - https://arks.itch.io/ps4-buttons <br/>
Base template created by DarknessFX - https://github.com/DarknessFX/UEPlaystationGamepad <br/>
DualShock4 v1 (GEN1) contribution and 5.0.3 port - https://github.com/Equ1no0x

## License

License: <a href="http://creativecommons.org/publicdomain/zero/1.0/" target="_blank">(Creative Commons Zero, CC0)</a>
<br/>
You can use this content for personal, educational, and commercial purposes.

##

DarknessFX
<br/>
• Website <a href="https://dfx.lv" target="_blank">https://dfx.lv</a><br/>
• Twitter <a href="https://twitter.com/DrkFX" target="_blank">DrkFX</a><br/><br/>
Equ1no0x
<br/>
• Website <a href="https://noxanimdev.itch.io/" target="_blank">https://noxanimdev.itch.io/</a><br/>
• Twitter <a href="https://twitter.com/Equ1no0x" target="_blank">Equ1no0x</a><br/>
