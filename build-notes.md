# Build Notes

---

# Packaging

In UE, open **Platforms** dropdown menu, select target platform, then "Package Product". A dialogue will ask for destination directory.

# Visual Studio

## NuGet

NuGet is a build in VisualStudio package manager. 

Helpful packages are:

- **Microsoft.DXSDK.D3DX**
	- This package contains the includes, import libraries, and DLLs required to use the legacy DirectX SDK's D3DX9, D3DX10, and/or D3DX11 utility libraries

- **Microsoft.XAudio2.Redist**
	- XAudio 2.9 redistributable for Windows 7 SP1, Windows 8, Windows 8.1, Windows 10 and Windows 11.

## DirectX SDK

After installing DXSDK, may need to update *Project > Properties > VC++ Directories*, with environment variables referenced in [Where is the DirectX SDK?](https://learn.microsoft.com/en-us/windows/win32/directx-sdk--august-2009-?redirectedfrom=MSDN.) **Note:** Make sure add the includes in the beginning of the list. 

1. Open **Properties** for the project and select the **VC++ Directories** page. ii. Select **All Configurations and All Platforms**. iii. Set these directories as follows:
  1. Include Directories:` $(IncludePath);$(DXSDK_DIR)Include`
  2. Library Directories: `$(LibraryPath);$(DXSDK_DIR)Lib\x86`
2. Click Apply.
  1. Choose the x64 Platform.
  2. Set the Library Directory as follows: `$(LibraryPath);$(DXSDK_DIR)Lib\x64`

3. Wherever `d3dx9.h`, `d3dx10.h`, or `d3dx11.h` are included in your project, be sure to explicitly include `d3d9.h`, `d3d10.h` and `dxgi.h`, or `d3d11.h` and `dxgi.h` first to ensure you are picking up the newer version. You can disable warning C4005 if needed; however, this warning indicates you are using the older version of these headers.