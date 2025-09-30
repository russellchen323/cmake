# ℹ️ Overview
How to compile across languages and platforms has been a long-standing problem.

Compiling cross-languages and cross-platform are significant challenge cuz compilers and language semantics differ. CMake is a cross-platform, automated build system that helps address this issue. By completing this project, users will learn the compiler workflow and principles behind static and dynamic libraries. The project includes step-by-step instructions for using CMake to build the handwritten number recognition system.


# 💡Concept

CMake main concept: Configrature :arrow_right: Generation :arrow_right: Built

<p align="center">
  <img width="580" height="460" src="https://github.com/russellchen323/cmake-tutorial/blob/main/CMake's%20config_gener_built.jpg">
</p>


## :one: Model desgin

|  Library  | Languages  |
|:----------|:-----------|
|onnxruntime| C++, Python|
|libpng     | C          |
|zlib       | C          |

This project will provide a recognize command-line tool and implement a handwritten number recognition library.
Users can call this command-line tool to recognize handwritten numbers for images.
Therefore, two build targets need to be defined in the project: one is the dynamic library target called `num_recognizer`, and the other is the executable target called recognize.
In addition, we hope to build the handwritten number recognition library as a general-purpose library that can be used anywhere, and even with languages other than C++.
In other words, the Application Binary Interface (ABI) should only share the Application Programming Interface (API). When the number recognition library exports its API, the exposed interfaces must only support the C language; function parameters and return values must be data types supported by C.


## :two: Folder of structure
The directory structure of the project is shown as below, please duplicate folder structure:
```python
└── cmake_for_onnxruntime_feature/
    ├── CMakeLists.txt #CMake file for main project
    ├── include/ #Header file of directory
    │   └── num_recognizer.h #Handwriting file of header file
    ├── src/ #Source of directory for handwriting
    │   └── num_recognizer.cpp #Source of file for handwriting
    ├── cli/ #Source of directory for command line tools
    │   └── recognize.c #Source of file for command line tools
    ├── models/ #handwriting model
    │   ├── .
    │   ├── .
    │   └── mnist.onnx
    ├── image/ #handwriting image
    │   ├── .
    │   ├── .
    │   ├── 8.png
    │   └── 9.png
    ├── Findonnxruntime/ #one of them source code
    │   ├── CMakeLists.txt #CMake file for source code of onnxruntime
    │   ├── main.cpp #main file for source code of onnxruntime
    │   └── build/ #onnxruntime source code have compiled libraries
    ├── cmake/ #user-defined cmake model
    │   ├── .
    │   ├── .
    │   ├── Findlibpng.cmake
    │   └── Findonnxruntime.cmake
    └── backup/ #have compiled libraries
        ├── libpng16.dll
        ├── onnxruntime.dll
        └── z.dll
```
> The $${\color{red}mnist.onnx}$$ neural network model file used here can be downloaded from the `onnx/models` code repository on GitHub. By the way, it is command tools in folder is called `cli`, the abbreviation is from commandline interface.

Before moving on to implementation, it is necessary to clearly define what functionalities which handwritten number recognition library should provide and design the interface accordingly. As a practical case, this library will not involve overly complex techniques. Nevertheless, author hopes that the handwritten number recognition library will still be practical: it should support both recognizing from a binarized image pixel array provided by the user and from a PNG image file path. In other words, though small, it will be fully functional!


## :three: Built third-party libraries

### :three:.:one: Install source of onnxruntime
In this project, the version of number adopted `1.22.1` , which accrodding on version dowloanded.

- This site was built using [onnxruntime](https://github.com/microsoft/onnxruntime/tags/).
- Chose a version, example `v1.22.1`, click link of `v1.22.1`.
- Rolled down webside, chose and downloaded `onnxruntime-win-x64-1.22.0.zip`.
- Please put unziped package on folder of current project, `cmake_for_onnxruntime_feature/Findonnxruntime`.


**Code lists-1: cmake_for_onnxruntime_feature/Findonnxruntime/Findonnxruntime.cmake (1-3line)**

```cmake
find_path(onnxruntime_INCLUDE_DIR onnxruntime_c_api.h
  HINTS $ENV{onnxruntime_ROOT}
  PATH_SUFFIXES "include")
```
The pakege of `onnxruntime` have two folders, one of them is `include/` of folder, anther of them is `lib/` of folder. The former has header of files, the latter has library of files.
Let's go to make a find of model!
Firstly, command is used `find_path` for look for where is `onnxruntime_c_api.h`, like as above **Code lists-1**.
It' used environment variable by `onnxruntime_ROOT` as candidate of path parameter. In addtional, `inclde/` is set as sub-folder. Result is saved on variable of `onnxruntime_INCLUDE_DIR` by `find_path` have found.


**Code lists-2: cmake_for_onnxruntime_feature/Findonnxruntime/Findonnxruntime.cmake (5-6line)**

```cmake
find_library(onnxruntime_LIBRARY
  NAME onnxruntime
  HINTS $ENV{onnxruntime_ROOT}
  PATH_SUFFIXES "lib")
```
And then it look for library of path of onnxruntime by `find_path`.
`onnxruntime_ROOT` which environment variable points to the root directory of onnruntime. In addtional, `inclde/` is set as sub-folder. result is saved on variable of `onnxruntime_LIBRARY` by `find_path` have found, like as above **Code lists-2**.


**Code lists-3: cmake_for_onnxruntime_feature/Findonnxruntime/Findonnxruntime.cmake (10-13line)**

```cmake
include(FindPackageHandleStandardArgs)

find_package_handle_standard_args(onnxruntime
  REQUIRED_VARS onnxruntime_LIBRARY onnxruntime_INCLUDE_DIR)
```
All necessary information have get, it's time to use model of `FindPackageHandleStandardArgs`, after checking information! it used by `find_package_handle_standard_args` to check its, do path of variable correct?


**Code lists-4: cmake_for_onnxruntime_feature/Findonnxruntime/Findonnxruntime.cmake (15-28line)**

```cmake
if(onnxruntime_FOUND)
  set(onnxruntime_INCLUDE_DIRS ${onnxruntime_INCLUDE_DIR})
  set(onnxruntime_LIBRARIES ${onnxruntime_LIBRARY})

  add_library(onnxruntime::onnxruntime SHARED IMPORTED)
  target_include_directories(onnxruntime::onnxruntime INTERFACE ${onnxruntime_INCLUDE_DIRS})
  if(WIN32)
    set_target_properties(onnxruntime::onnxruntime PROPERTIES
      IMPORTED_IMPLIB "${onnxruntime_LIBRARY}")
  else()
    set_target_properties(onnxruntime::onnxruntime PROPERTIES
      IMPORTED_LOCATION "${onnxruntime_LIBRARY}")
  endif()
endif()
```
It's time to use `onnxruntime_FOUND` of variable, it have found successfully so it's be true. Finally, it will creat an imported libraries of object by `onnxruntime::onnxruntime`, like as above **Code lists-5**.
In CMake, `IMPORTED_LOCATION` specifies the path to the actual library file (like a `.dll` or `.so`) for an imported target, while `IMPORTED_IMPLIB` (used on Windows) specifies path to import library (`.lib` file) that is paired with a DLL for linking purposes. User set `IMPORTED_LOCATION` to the primary library file for linking, and `IMPORTED_IMPLIB` to the corresponding `.lib` file on Windows when linking against a DLL. 


**Code lists-5: cmake_for_onnxruntime_feature/Findonnxruntime/CMakeLists.txt**

```cmake
cmake_minimum_required(VERSION 3.20)
project(find-onnxruntime)
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR};${CMAKE_MODULE_PATH}")
set(CMAKE_CXX_STANDARD 17)
set(onnx_version 1.22.1)

if("$ENV{onnxruntime_ROOT}" STREQUAL "")
  if(WIN32)
	  set(ENV{onnxruntime_ROOT} "${CMAKE_CURRENT_LIST_DIR}/onnxruntime-win-x64-${onnx_version}")
  elseif(APPLE)
      set(ENV{onnxruntime_ROOT} "${CMAKE_CURRENT_LIST_DIR}/onnxruntime-osx-universal2-${onnx_version}")
  else()
      set(ENV{onnxruntime_ROOT} "${CMAKE_CURRENT_LIST_DIR}/onnxruntime-linux-x64-${onnx_version}")
  endif()
endif()

find_package(onnxruntime 1.10)
add_executable(main main.cpp)
target_link_libraries(main onnxruntime::onnxruntime)
target_compile_definitions(main PRIVATE ORT_NO_EXCEPTIONS)
```
Directory program insert current directory to `CMAKE_MODULE_PATH` of variable for just moment write found-model file, and then it set `onnxruntime_ROOT` environment as root of directory. And then call command of `find_package` for looking for library of `onnxruntime`, finially, creat a excuative file of object, and then connect `onnxruntime::onnxruntime` of this library of object.
In **Code lists-6**, it is defined `MACRO` of `ORT_NO_EXCEPTIONS` for executive files of object, it is meaning that easy to demonstrate, when it show error at `onnxruntime` have happened error. 


**Code lists-6: cmake_for_onnxruntime_feature/Findonnxruntime/main.cpp**

```cmake
#include <onnxruntime_cxx_api.h>

int main() {
    Ort::Env env;
	Ort::Session session(env, ORT_TSTR(""), Ort::SessionOptions(nullptr));
	return 0;
}
```


**Implement command**

```bash
cd cmake_for_onnxruntime_feature/Findonnxruntime
mkdir build; cd build
cmake ..
cmake --build .
```
User could have tried executive built files, but user will see message of error which is load of model isn't exist. However, it is proved that `onnxruntime` successfully have connected on program of main. Please remember dynamic of library, `onnxruntime.dll` duplicate to folder layer with `main.exe`.


### :three:.:two: Install source of libpng
- Use could have got `libpng` library from its GitHub repository, cloning source code and install it.
- In this project, the version of number adopted `1.6.40`.

It have install `onnxruntime` successfully, let's pay a attention other third-party libraries which are `zlib` and `libpng` of libraries, right now. Let's go head install `libpng`!

**Implement command**
```bash
cd libpng
mkdir build; cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build . --config Release
cmake --install .
```
Library of `libpng` bring command `find_package` of configuration files which is configurture model by oneself, but lose related header of director etc. Therefore, do packege twice, wrote a found-model file by user-defined that is named as `<Find-library_name>`. The found-model file core as below. Keep in mind, user should type command on administrator of autherither environment.

**Code lists-7: cmake_for_onnxruntime_feature/cmake/Findlibpng.cmake**

```cmake
# Configuration files is used which libpng is itslef.
find_package(libpng CONFIG CONFIGS libpng16.cmake)

# System is going setting a varifiy which is <library_name>_FOUND when find_package of function was used.
if(libpng_FOUND)
	# Firstly, It try to set property of IMPORTED_LOCATION for obtained dynamic of path.
    get_target_property(libpng_LIBRARY png_shared IMPORTED_LOCATION)
	# It try to set property of IMPORTED_LOCATION_RELEASE, if couldn't get dynamic of path.
    if(NOT libpng_LIBRARY)
        get_target_property(libpng_LIBRARY png_shared IMPORTED_LOCATION_RELEASE)
    endif()

    # It setting root of directory, according png dynamic of path.
    set(_png_root "${libpng_LIBRARY}/../..")

    # Looking for the directory path where the png.h header file is folder.
    find_path(libpng_INCLUDE_DIR png.h
        HINTS ${_png_root}
        PATH_SUFFIXES include/libpng16 include
    )

    # Target is assigned to directory of include.
    # Tell CMake, static of library or dynamic of library is used, those will automaticly connect to "${libpng_INCLUDE_DIR}".
    if(libpng_INCLUDE_DIR)
        target_include_directories(png_shared INTERFACE ${libpng_INCLUDE_DIR})
        target_include_directories(png_static INTERFACE ${libpng_INCLUDE_DIR})
    endif()
endif()

include(FindPackageHandleStandardArgs)

# Do verificated variables exist.
find_package_handle_standard_args(libpng
    REQUIRED_VARS libpng_LIBRARY libpng_INCLUDE_DIR
    CONFIG_MODE
)

# Result of variables is setting.
if(libpng_FOUND)
    set(libpng_INCLUDE_DIRS ${libpng_INCLUDE_DIR})
    set(libpng_LIBRARIES ${libpng_LIBRARY})
endif()
```
Please try to combine official document and understand above by yourself.


### :three:.:three: Install source of zlib
- Use could have got `zlib` library from its GitHub repository, cloning source code and install it.
- In this project, the version of number adopted `v1.3.1`

**Implement command**
```cmake
cd zlib
mkdir build; cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build . --config Release
cmake --install .
```
Library of `zlib` could have got from its GitHub repository. Please clone it to local, before up to as above steps for install it.
CMake will install default directory of system, when type in command `cmake --install` for install CMake iteam. In Windows system, general path is `C:\Program Files (x86)\zlib` in system. In contrast, in Linux system, general path is `/usr/local`. Keep in mind, user should type command on administrator of autherither environment, if would like to use default install of path. Off course, user could have specified `--prefix <install director>` for command of `cmake --install`, could have user-defined installation directory. And then looking for third-party of libraries by command of `find_package`. user usually need to have manually specified and promptly install directories and variable.


**Code lists-8: Findzlib.cmake**

Please observer current folder structure and achieve mastery through a comprehensive study of contents which had introduced ever files, before will build `Findzlib.cmake` of file by oneself.


## :four: Built libraries for handwritten number recognition

It's time to construct code of structure and comprehensive infrastructure in preferred order.

### :four:.:one: Built a raw-code of C++ for main project

Firstly, user should define a couple of raw-code with global variable and local variable etc., cut the crap, let's do it!

**Code lists-9: cmake_for_onnxruntime_feature/src/num_recognizer.cpp (21-39line)**

```cpp
// Model input parameter name.
static const char *INPUT_NAMES [] = {"Input3"};
// Model output parameter name.
static const char *OUTPUT_NAMES[] = {"Plus214_Output_0"};
// Model input image width.
static constexpr int64_t INPUT_WIDTH = 28;
// Model input image height.
static constexpr int64_t INPUT_HEIGHT = 28;
// The dimension of the input data.
static const std::array<int64_t, 4> input_shape{
	1, 1, INPUT_WIDTH, INPUT_HEIGHT};
// The dimension of the output data.
static const std::array<int64_t, 2> output_shape{
	1,10};

// Onnxruntime environment
static Ort::Env env{nullptr};
// Onnxruntime cache memory information
static Ort::MemoryInfo memory_info{nullptr};
```

In while, costant is defined by mnist of model that one of neural networks, like as above topic of **:two: Folder of structure**. user should have modified corresponding parameter, if would like to swap other mnist of model that another of neural networks.
On the other hand, varible is from `onnxruntime` of eniviroment due to build it with static initial, temporarily don't define its staus(I.e. used nullptr for initialization). it will export to function of `num_recognizer_init`.


**Code lists-10: cmake_for_onnxruntime_feature/src/num_recognizer.cpp (42-46line)**

```cpp
//! @brief handwritten number recognition
struct Recognizer {
//! @brief onnruntimetime session
Ort::Session session;
};
```
It defined a class for handwritten number recognition, and encapsulated `onnxruntime` session. ever handwritten number recognitions corresponding one of session, ever sessions corresponding one of `onnxruntime` of model.


**Code lists-11: cmake_for_onnxruntime_feature/src/num_recognizer.cpp (87-91line)**

```cpp
extern "C" {
void num_recognizer_init() {
	env = Ort::Env{static_cast<const OrtThreadingOptions *>(nullptr)};
	memory_info = Ort::MemoryInfo::CreateCpu(OrtDeviceAllocator, OrtMemTypeCPU);
}
```
It's the first function to be implemented for initialization interface, it's used `extern C` to be defined part of C language, as above. Actually, initialization interface is inital two global varible with related `onnxruntime` whom are env and `memory_info`.


**Code lists-12: cmake_for_onnxruntime_feature/src/num_recognizer.cpp (95-107line)**

```cpp
void num_recognizer_create(const char *model_path, Recognizer **out_recognizer) {
	Ort::Session session{nullptr};
#if _WIN32
	// In Windows system, the onnxruntime session must use const when accepting the model file path.
	// wchar_t*, that is, wide character string, so do convert in here.
	wchar_t wpath[256];
	MultiByteToWideChar(CP_ACP, MB_PRECOMPOSED, model_path, -1, wpath, 256);
	session = Ort::Session(env, wpath, Ort::SessionOptions(nullptr));
#else
	session = Ort::Session(env, model_path, Ort::SessionOptions(nullptr));
#endif
	*out_recognizer = new Recognizer{std::move(session)};
}
```
Here, main logic just build serious of session by model of `onnxruntime`, and will assign value for created class whom handwritten number recognition, and then its class will as pointer result and transfer to user.


**Code lists-13: cmake_for_onnxruntime_feature/src/num_recognizer.cpp (13-18line)**

```cpp
#ifdef _WIN32
// In the Windows operating system, 
// we need to use the Windows API to help complete the conversion from const char* to const.
// wchar_t* encoding conversion. Therefore, it is necessary to reference windows.h.
#include <Windows.h>
#endif
```
It should have handled a trouble here, would like to covert encode API on Windows, user should include `Windows.h`, cuz model encode is difference when session is be constructed, so imported parameters need to have encoded convert.


**Code lists-14: cmake_for_onnxruntime_feature/src/num_recognizer.cpp (110line)**

```cpp
void num_recognizer_delete(Recognizer *recognizer) { delete recognizer; }
```
The delete of structure was siample as above.


**Code lists-15: cmake_for_onnxruntime_feature/src/num_recognizer.cpp (113-132line)**

```cpp
int num_recognizer_recognize(Recognizer *recognizer, float *input_image,
							int *result) {
	std::array<float, 10> results{};

	auto input_tensor = Ort::Value::CreateTensor<float>(
	memory_info, input_image, INPUT_WIDTH * INPUT_HEIGHT,
	input_shape.data(), input_shape.size());
	
	auto output_tensor = Ort::Value::CreateTensor<float>(
	memory_info, results.data(), results.size(), output_shape.data(),
	output_shape.size());
	
	recognizer->session.Run(Ort::RunOptions(nullptr), INPUT_NAMES,
							&input_tensor, 1, OUTPUT_NAMES, &output_tensor, 1);

	*result = static_cast<int> (std::distance(
	results.begin(), std::max_element(results.begin(), results. end())));

	return 0;
}
```
It use mnist of model that neural networks could have accepted 28 x 28 of float type as input, and then distribute result of output from 0 to 10 weight.


**Code lists-16: cmake_for_onnxruntime_feature/src/num_recognizer.cpp (135-206line)**

```cpp
int num_recognizer_recognize_png(Recognizer *recognizer, const char *png_path, int *result){
	
	int ret = 0;
	std::array<float, INPUT_WIDTH * INPUT_HEIGHT> input_image;
	FILE *fp;
	unsigned char header[8];
	png_structp png_ptr;
	png_infop info_ptr;
	png_uint_32 png_width, png_height;
	png_byte color_type;
	png_bytep *png_data;
	
	// Open the PNG image file.
	fp = fopen(png_path, "rb");
	if (!fp) {
		ret = -2;
		goto exit3;
	}
	
	// Read PNG image file header.
	fread(header, 1, 8, fp);
	// Verified file of header wether is a PNG format file header.
	if(png_sig_cmp (reinterpret_cast<unsigned char *> (header), 0, 8)) {
	ret = -3;
	goto exit2;
	}
	
	// Create a PNG pointer data structure.
	png_ptr = png_create_read_struct(PNG_LIBPNG_VER_STRING, nullptr, nullptr, nullptr);
	if (!png_ptr) {
	ret = -4;
	goto exit2;
	}
	
	// Create PNG information pointer data structure.
	info_ptr = png_create_info_struct(png_ptr);
	if (!info_ptr) {
		ret = -5;
		goto exit2;
	}
	
	// Set up jumps, if handle is exceptions.
	if (setjmp(png_jmpbuf(png_ptr))) {
	ret = -6;
	goto exit2;
	}
	
	// Initialize PNG file.
	png_init_io(png_ptr, fp);
	png_set_sig_bytes(png_ptr, 8);
	
	// Read PNG information
	png_read_info(png_ptr, info_ptr);
	// PNG image width
	png_width = png_get_image_width(png_ptr, info_ptr);
	// PNG image height
	png_height = png_get_image_height(png_ptr, info_ptr);
	// PNG image color type
	color_type = png_get_color_type(png_ptr, info_ptr);
	
	// Set up jumps, if handle is exceptions.
	if (setjmp(png_jmpbuf(png_ptr))) {
	ret = -7;
	goto exit2;
	}
	
	// Read PNG information
	png_data = (png_bytep *)malloc(sizeof(png_bytep) *png_height);
	for (unsigned int y = 0; y < png_height; ++y) {
		png_data [y] = (png_byte *)malloc(png_get_rowbytes(png_ptr, info_ptr));
	}
	png_read_image(png_ptr, png_data);
```
It's key point on recongized png image that how will png of image transfer to 28 x 28 of binarization of image in array.
Firstly, it read PNG of image by third party library which is `libpng` compeleted. 
The function begins by opening specified PNG file in binary mode and verifying its header to ensure that it is a valid PNG format. Once validated, it creates and initializes the necessary `libpng` structures for decoding. Error handling is established through setjmp to safely recover from any `libpng` runtime errors. After initializing file I/O, the function reads the image metadata, including width, height, and color type. Memory is then allocated to hold raw pixel data, with each row allocated according to row size reported by `libpng`. Finally, image contents are fully read into the `png_data` buffer, which now contains the decoded pixel values ready for further processing, normalization, and feeding into the recognizer.


**Code lists-17: cmake_for_onnxruntime_feature/src/num_recognizer.cpp (49-85line)**

```cpp
//! @brief convert the byte type color value to the binary Eloat type value accepted by the model
//! @param b color value of byte type
//! @return the binary float type value accepted by the model, 0 as white, 1 as black.
static float byte2float(png_byte b) { return b < 128 ? 1: 0; }
//! @brief get the float type color value of the specified pixel after binary conversion of the PNG image
//! @param x x pixel horizontal coordinate
//! @param y y pixel horizontal coordinate
//! @param png_width png_width image width
//! @param png_height png_width image height
//! @param color_type image color type
//! @param png_data png of image data
//! @return the float type color value of the corresponding pixel after binarization

static float get_float_color_in_png(
    unsigned int x,
    unsigned int y,
    png_uint_32 png_width,
    png_uint_32 png_height,
    png_byte color_type,
    png_bytepp png_data
) {
    if (x >= png_width || x < 0) return 0;
    if (y >= png_height|| y < 0) return 0;

    switch (color_type) {
    case PNG_COLOR_TYPE_RGB: {
        auto p = png_data[y] + x * 3;
        return byte2float((p[0] + p[1] + p[2]) / 3);
    } break;
    case PNG_COLOR_TYPE_RGBA: {
        auto p = png_data[y] + x * 4;
        return byte2float((p[0] + p[1] + p[2] + p[3]) / 3);
    } break;
    default:
        return 0;
    }
}
```
Building on this, the `get_float_color_in_png` function retrieves normalized float value of a specific pixel (x, y) from decoded PNG buffer. It supports both RGB and RGBA formats by averaging relevant channel values ​​before passing them through `byte2float`. Out-of-bound coordinates or unsupported color types return 0 by default.


**Code lists-18: cmake_for_onnxruntime_feature/src/num_recognizer.cpp (209-229line)**

```cpp
// PNG image is resample and scale it should have fitted input image size and then is accepted by the model.
	for(unsigned int y = 0; y < INPUT_HEIGHT; ++y) {
		for(unsigned int x = 0; x < INPUT_WIDTH; ++x) {
			float res = 0;
			int n = 0;
			for (unsigned int png_y = y * png_height / INPUT_HEIGHT;
				png_y < (y + 1) * png_height / INPUT_HEIGHT; ++png_y) {
				for (unsigned int png_x = x * png_width / INPUT_WIDTH;
					png_x < (x + 1) * png_width / INPUT_WIDTH; ++png_x) {
					res += get_float_color_in_png(png_x, png_y, png_width,
												  png_height, color_type, 
												  png_data);
					++n;
				}
			}
			input_image[y * INPUT_HEIGHT + x] = res / n;
		}
	}
	
	// Recognized handwritten number in image data.
	ret = num_recognizer_recognize(recognizer, input_image.data(), result);
```
After decoding PNG and defining pixel conversion helpers, PNG image is resampled and scaled to match model’s expected input size (`INPUT_WIDTH` × `INPUT_HEIGHT`). For each output pixel in resized image grid, code determines corresponding rectangular region in original PNG.
Once the entire image has been resampled and binarized, the resulting `input_image` array contains the model-ready representation of the PNG. This processed data is passed to recognizer through `num_recognizer_recognize`, which produces the final recognition result and stores it in result.


**Code lists-19: cmake_for_onnxruntime_feature/src/num_recognizer.cpp (232-245line)**

```cpp
	exit1:
		// Released the memory space storing PNG image data.
		for (unsigned int y = 0; y < png_height; ++y) {
			free(png_data[y]);
		}
		free(png_data);
		
	exit2:
		// Close files.
		fclose(fp);

	exit3:
	return ret;
	}
```
To ensure proper memory and file management, function uses labeled exit points combined with goto for structured cleanup. If the function reaches `exit1`, it iterates through all allocated rows of `png_data`, frees each one, and then frees row pointer array itself. At `exit2`, it closes file pointer associated with the PNG file, preventing resource leaks. Finally, at `exit3`, function returns status code ret, which indicates either success or the specific error that occurred during processing, such that structured cleanup guarantees that all allocated resources are correctly released.


### :four:.:two: Built a header of C++ for main project

Second, when interface is designing, actually, it wrote core part of a header file for interface functions. However, it's not enough complete public file, if just simply declaring this function. In addition, it is adopted exposed C language as handwritten number recognition library, that header file is usable by both C++ and C programs. Let’s take a look at how to implement this header file.

It is first necessary to initialize an `onnxruntime` environment `(Ort::Env)` for the `onnxruntime` session `(Ort::Session)` to use, when using the `onnxruntime` library. Therefore, the handwritten number recognition library should first provide an initialization interface, as shown below.


**Code lists-20: cmake_for_onnxruntime_feature/include/num_recognizer.h (1-4line)**

```cpp
#ifndef NUM_RECOGNIZER_H
#define NUM_RECOGNIZER_H

#include "num_recognizer_export.h"

```
Firstly, it include header of file, such that could have used micro of `num_recognizer_EXPORT` etc.


**Code lists-21: cmake_for_onnxruntime_feature/include/num_recognizer.h (7-16line)**

```cpp
#ifdef __cplusplus
extern "C" {
#endif

#ifdef num_recognizer_EXPORTS
struct Recognizer;
#else
// Predeclaration
typedef struct _Recognizer Recognizer;
#endif
```
Second, it's using C++ of compiler, due to interface functions are C language. Structures and function declarations related to the interface need to have enclosed with `extern "C"`. Here, macro of `__cplusplus` is used to determine whether a C++ compiler is being used. Meanwhile, struct have decleared.


**Code lists-22: cmake_for_onnxruntime_feature/include/num_recognizer.h (19-43line)**

```cpp
//! @brief initialize the handwriting recognition library
NUM_RECOGNIZER_EXPORT void num_recognizer_init();

//! @brief creating a Recognizer
//! @param model_path model file path
//! @param[out] out_recognizer initialized identifier pointer is accepted
NUM_RECOGNIZER_EXPORT void num_recognizer_create(const char *model_path, Recognizer **out_recognizer);

//! @brief destroy the identifier
//! @param recognizer pointer of identifier
NUM_RECOGNIZER_EXPORT void num_recognizer_delete(Recognizer *recognizer);

//! @brief recognized handwritten numbers in PNG images
//! @param recognizer pointer of identifier
//! @param input_image model input (28x28 float, 0=white, 1=black)
//! @param result recognition results of pointer have received
//! @return Successful=0
NUM_RECOGNIZER_EXPORT int num_recognizer_recognize(Recognizer *recognizer, float *input_image, int *result);

//! @brief recognized handwritten number in PNG images
//! @param recognizer pointer of identifier
//! @param png_path path is png 
//! @param result recognition results of pointer have received
//! @return Successful=0
NUM_RECOGNIZER_EXPORT int num_recognizer_recognize_png(Recognizer *recognizer, const char *png_path, int *result);
```
There involved two cases here, header file is implemented on raw-file of reference (I.e., `num_recognizer.cpp`) and refered on external program of user (source file of recognize command-line tool will be written.
The `num_recognizer_EXPORTS` macro can be used to determine whether the library is being built or being used. Remember? It was defined in the project directory shown in **Code lists-27**. It means library is currently being built, when this macro is defined. `Recognizer` structure is forward-declared to ensure the compiler recognizes it when declaring the interface functions at this time. The actual definition of this structure will be provided in source file.
If macro isn't used that meaning is library using by user, it don't have include forward-declared only at this time, otherwise report will show couldn't find definition of error message from compiler. It will need to define struct of `Recognizer` as opaque structure, I.e. it don't define sepcified struct which only appear in pointer. it conform to classing `Recognize` of situation in interface.


**Code lists-23: cmake_for_onnxruntime_feature/include/num_recognizer.h (46-50line)**

```cpp
#ifdef __cplusplus
} // extern "C"
#endif

#endif // NUM_RECOGNIZER_H
```
Finally, please remember front model of `extern "C"`, and close preprocessor which is `#if`.


### :four:.:three: Built a command line of tools for main project

All files is complete so much, right now. Don't matter either user adopts C, C++ of language, or built handwritten number recognition, or distributed to user used, it is working.
Source code have completed, go ahead to finish write command tools. C language is general program language which show its charm , adopted C in here, nor C++. it will call library of C++, such that futher checking whether built handwritten number recognition is complete or not.

**Code lists-24: cmake_for_onnxruntime_feature/cli/recognize.c**
```cpp
#include <stdio.h>
#include <num_recognizer.h>

//! @brief main function
//!
//! There should be 3 command line parameters:
//! 1. The file name of the command line program itself, I.e. recognize.
//! 2. Model file path.
//!	3. Path PNG image file will have recognized.
//!
//! E.g. recognize models/mnist.oonx 2.png.

//!
//! @param argc number of command line parameters
//! @param argv An array of command line argument values
//! @return return code, 0 as normal exit
int main (int argc, const char **argv) {
	// Is number of command line parameters as 3.
	if (argc != 3) {
	printf("Usage: recognize mnist.onnx 3.png\n");
	// Return error code -1.
	return -1;
	}

	// Return code.
	int ret = 0;
	// Recognition results.
	int result = -1;
	// Identifier of pointer.
	Recognizer *recognizer;
	// Initialize the recognizer.
	num_recognizer_init();
	
	// Use the model file to create a recognizer, argv[1] is the model file path.
	num_recognizer_create(argv[1], &recognizer);
	
	// Recognize handwritten numbers in image files, argv[2] is the image file path.
	if (ret = num_recognizer_recognize_png(recognizer, argv[2], &result)) {
		// The return value is not 0, and an error occurred during the recognition process.
		printf("Failed to recognize\n");
		goto exit_main;
	}
	// Print recognition results.
	printf("%d\n", result);

exit_main:
	// Destructor Identifier.
	num_recognizer_delete(recognizer);
 	// Returns a normal exit return code of 0.
	return ret;
}
```
Command line tools is easy as above.


## :five: Built CMake directory

Prepare work have done finally! now, go ahead to write a part which library is handwritten number recognition. Fitstly, well done built a `CMakeLists.txt` and according with convention within  root of directory, as below.

### :five:.:one: Built a CMake file for main project

**In this part, it's most key point on this project**, should create a `CMakeLists.txt` file to integrate `onnxrumtime`, `libpng`, `zlib`, and the user-defined library of handwriting number recognition for compile, and link all the libraries.

**Code lists-25: cmake_for_onnxruntime_feature/CMakeLists.txt (1-6line)**

```cmake
cmake_minimum_required(VERSION 3.20)
project(num_recognizer)
# "CMAKE_MODULE_PATH" should have defined due to use find_package of module.
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
# Compiler standard is defined as 11.
set(CMAKE_CXX_STANDARD 17)
```
Fistly, base information set as above, it should declear `CMAKE_MODULE_PATH` that where is model of cmake such that cmake could have found thrid-part libraries.

**Code lists-26: cmake_for_onnxruntime_feature/CMakeLists.txt (9-28line)**

```cmake
set(onnx_version 1.22.1)

# Please put compressed package of library of onnxruntime on project.
# The following environment variable settings are for demonstration purposes only.
# In actual development of path, you shouldn't up to the defaultive dependent installation of directories in the hard-code.
if("$ENV{onnxruntime_ROOT}" STREQUAL "")
	if(WIN32)
	set(ENV{onnxruntime_ROOT} "${CMAKE_CURRENT_LIST_DIR}/Findonnxruntime/onnxruntime-win-x64-${onnx_version}")
elseif(APPLE)
	set(ENV{onnxruntime_ROOT} "${CMAKE_CURRENT_LIST_DIR}/Findonnxruntime/onnxruntime-osx-universa12-${onn_version}")
else()
	set(ENV{onnxruntime_ROOT} "${CMAKE_CURRENT_LIST_DIR}/Findonnxruntime/onnxruntime-linux-x64-${onnx_version}")
	endif()
endif()

# Specified minimum of version.
find_package(onnxruntime 1.10 REQUIRED)

# Look for library of libpng.
find_package(libpng REQUIRED)
```

It should have set environment of variable which is `onnxruntime ROOT` for looking for `onnxruntime` library. environment of variable will dependence difference systems, such as Windows, macOS, and Linux. In this point, isn't it interesting by CMake?


**Code lists-27: cmake_for_onnxruntime_feature/CMakeLists.txt (31-49line)**

```cmake
add_library(num_recognizer SHARED src/num_recognizer.cpp)

include(GenerateExportHeader)
generate_export_header(num_recognizer)
set_target_properties(num_recognizer PROPERTIES
	CXX_VISIBILITY_PRESET hidden
	VISIBILITY_INLINES_HIDDEN 1
)

# Look for all files which are on folder inlcude.
target_include_directories(num_recognizer PUBLIC include ${CMAKE_BINARY_DIR})




# Linked each libraries.
target_link_libraries(num_recognizer PRIVATE onnxruntime::onnxruntime png_shared)
# Set compile definitions.
target_compile_definitions(num_recognizer PRIVATE ORT_NO_EXCEPTIONS num_recognizer_EXPORTS)
```

In addition, it invoking `add_library` command to create a dynamic library of built target, the `GenerateExportHeader` module is also referenced, such that its `generate_export_header` command is used generate export header file for the dynamic library. At the same time, two properties of library target are setting hide symbols by default and only export explicitly specified symbols.

To allow library to directly include newly generated export header file, current binary directory `${CMAKE_BINARY_DIR}` must have added library target’s header search path. In addition, the `num_recognizer_EXPORTS` macro is defined for library to indicate that it is being built rather than used, ensuring that macro definitions in export header are correct.

The include directory is also added to the library’s header search of path, and `onnxruntime` and `libpng` etc., third-party library targets are linked to library target. In addtion, it are packaging a dynamic library that conforms to the C language ABI, and it don’t want exceptions to be thrown in the program, the `ORT_NO_EXCEPTIONS` macro is defined to disable exceptions in `onnxruntime` library.


**Code lists-28: cmake_for_onnxruntime_feature/CMakeLists.txt (52-55line)**

```cmake
# Finially, added external of file which is recognize.c as enter point on project.
add_executable(recognize cli/recognize.c)
# Finally, added all executive files on recognize for generated an executive file.
target_link_libraries(recognize PRIVATE num_recognizer)
```

This CMake configuration creates recognize executable from the CLI source file and links it with the `num_recognizer` library. In doing so, it combines core recognition functionality with a command-line interface, producing a standalone tool that can run image recognition tasks directly from the terminal.


### :five:.:two: Compiled and ran CMakeLists.txt by MSVC

All have well done, is it excited to excute program? please up to as below!

- Use the [terminal](https://code.visualstudio.com/docs/terminal/basics/) on vscode to execute command.

**Implement command**

```cmake
cd cmake_for_onnxruntime_feature
mkdir build; cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build . --config Release
```

It have used MSVC for compiled, `recognize` should be in Release of folder, I.e., `\Release\recognize.exe`. In addition, please put files which as below on folder which is same layer with `recognize.exe`, before user will excute `recognize.exe` in Windows system.

- Put files of `backpup/onnxruntime.dll`, `backpup/libpng16.dll` and `backpup/z.dll` on folder of `/Release`


**Implement command**

```cmake
.\Release\recognize.exe ..\models\mnist-7.onnx ..\image\5.png
```
Verfication function is work by compilered file.


## :octocat: Result screenshot
- User can see that the file cross-compiled by cmake successfully recognizes the handwritten image.

![image](https://github.com/russellchen323/cmake-tutorial/blob/main/Result%20of%20image.jpg)


- To trying swap model, Is the recognition rate improved?


## :page_facing_up: Summary
This chapter uses CMake to organize the project structure and build process, introduces multiple third-party of libraries, and implements a complete and practical handwritten number recognition library project using C and C++. This chapter provides readers with a deeper understanding of CMake's capabilities and a better grasp of the complete process from design to implementation of C and C++ programs.


## 💭 Invite users to give feedback and contribute

If you found this guide insightful or if you have suggestions, please start a Discussion!

When making open source software, you share your work with the world. Whether that is in the hope of contributions back, humbly if just one other person out there finds it useful, or building a community, I think it is important to solicit engagement.

If you want to encourage others to contribute back to your project, this is the place to do it. Point people to your DEVELOPMENT and/or CONTRIBUTING guides if you have them. Further, you can outline any other ways to contribute such as translating the README or documentation.


## 📖 Further reading and reference

- https://cmake.org/documentation/
- https://cmake.org/cmake/help/latest/guide/tutorial/index.html
- https://www.youtube.com/watch?v=nlKcXPUJGwA
- https://www.ptpress.com.cn/shopping/buy?bookId=7e5d6c08-1a29-4d41-a495-c9ede4622794


## :information_source: Information

|  ✍️ Autor  |   :date: Day   |   :email: Email    |
|:-----------|:---------------|:-------------------|
|Russell Chen.|24st Sep, 2025.|RussellChen@ami.com.|


## :statue_of_liberty: Statement:
Copyright (c) American Megatrends Corporation.  All rights reserved.
Permission is granted to use, modify, merge, publish, distribute, sublicense, or sell the software and its documentation free of charge.
The copyright notice and this permission notice must be included in all copies or significant portions of the software.
Software is provided "as is" without any warranties, and the authors are not liable for any claims, damages, or liabilities.


> [!NOTE]
> Useful information that users should know, even when skimming content.

> [!TIP]
> Helpful advice for doing things better or more easily.

> [!IMPORTANT]
> Key information users need to know to achieve their goal.

> [!WARNING]
> Urgent info that needs immediate user attention to avoid problems.

> [!CAUTION]
> Advises about risks or negative outcomes of certain actions.
