To create a new ESP component, you can use the shortcut inside Visual Studio Code. Press `Ctrl` + `Shift` + `p`, and type `component` in the search bar. The option `ESP-IDF: Create New ESP-IDF Component` should show up. Select it. You are then prompted for the new component's name. Enter a descriptive name and press enter. After that, a new component should have been created inside the project under the `components` directory. You can then add your own logic in the generated C-file. Make sure that you update your header file for the newly created component inside your component's directory and then inside the `include` directory.
# Errors
You might get some errors referring to includes or to ESP-IDF/ADF libraries once you added some logic and tried to build the component. The reason why is probably because the component, by default, does not include many ESP-libraries you might be including in your C-file. To fix this, you can manually reference all of the required libraries in the `CMakeLists.txt` file **inside your component's directory, not inside the main component's directory**. The file would look something like this:
```
set(requires library1 library2)

idf_component_register(SRCS "my_component.c"
                    INCLUDE_DIRS "include"
                    REQUIRES ${requires})
```
> An includable library might be `bluetooth_service` or `esp_peripherals` for example.

However, if you are unsure which libraries to add, you can also add your entire `main` component to your `CMakeLists.txt` file. That would look like the following:
```
idf_component_register(SRCS "my_component.c"
                    INCLUDE_DIRS "include"
                    REQUIRES main)
```
> ==Warning==
> Including your entire `main` component is generally not recommended, especially when the other component uses only a small number of header files.