# IDF ADF Installation Guide

## Install IDF

### Using VSCode
Installation via the extension:
1. Search for the `ESP-IDF` extension and install it.
2. Open the extension (icon in left toolbar)
3. Select the `Advanced` installation option
4. Select the version that is stable and compatible with ADF: `release/v4.4`
5. Also install the `ESP-Tools` during installation

Manual installation:
1. Clone the `ESP-IDF` repository for the stable release version: `4.4`
Open a command line tool and change directory to the clone location using `cd %USERPROFILE%\esp`. (`mkdir %USERPROFILE%\esp` if it does not exist)
The repository will be stored in `%USERPROFILE%\esp\esp-idf` using this command.
Clone command (you need to have Git installed): `git clone "https://github.com/espressif/esp-idf.git" -b release/v4.4 --recursive`
2. Install the tools using the downloaded install script (Command Prompt):
`cd %USERPROFILE%\esp\esp-idf`
`install.bat esp32`
3. Install and open the VSCode extension and select installation from local repository.

#### Additional install locations on Windows
Useful for removing the current installation. Disclaimer: your location may be different than the default.
- VSCode extension folder: `%USERPROFILE%\.vscode\extensions` (you should uninstall the VSCode extension first)
- VSCode `ESP-IDF` extension downloaded tools: `%USERPROFILE%\.espressif`

### Using CLion
TODO

## Install ADF

### Using VSCode
1. Clone the ADF repository for the latest release version: `2.6`.
Open a command line tool and change directory to the clone location using `cd %USERPROFILE%\esp`. (`mkdir %USERPROFILE%\esp` if it does not exist)
The ESP-ADF repository will be stored in `%USERPROFILE%\esp\esp-adf` using this command.
Clone command (you need to have Git installed): `git clone "https://github.com/espressif/esp-adf.git" -b v2.6 --recursive`
2. After the cloning has finished (may take a while): open VSCode and search for the extension action `ESP-IDF: Install ESP-ADF` (using shortcut `F1`).
Choose `Use existing repository`,`GitHub`. When asked, choose the repository location as mentioned above.

### Using CLion
TODO
