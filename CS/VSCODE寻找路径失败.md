```json
{

"configurations": [

{

"name": "Linux", // 或 Mac, Win32

"includePath": [

"${workspaceFolder}/**",

"/usr/local/include/opencv4/**" // <--- 把你的路径加在这里

],

"defines": [],

"compilerPath": "/usr/bin/gcc", // 确保这里是你用的编译器

"cStandard": "c17",

"cppStandard": "c++17",

"intelliSenseMode": "linux-gcc-x64" // 根据你的系统选择

}

],

"version": 4

}
```

增添c_cpp_properties.json