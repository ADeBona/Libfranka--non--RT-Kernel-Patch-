# Libfranka Non-RT Patch

This repository provides instructions to manually apply a patch to the `libfranka` library, enabling it to run on non-realtime (Non-RT) kernels. The modifications allow users to bypass the strict requirement for a realtime kernel, which is beneficial for systems that cannot use RT kernels due to hardware or software constraints.

For further information about the original patch, please refer to the following GitHub repository: [Libfranka-Non-RT-Patch](https://github.com/heracleia/Libfranka-Non-RT-Patch).

## Why Use This Repository?

The modifications described here allow you to:

- Use `libfranka` on systems running non-realtime kernels.
- Avoid errors related to missing realtime kernel capabilities.
- Improve compatibility with a wider range of development environments.

## Manual Patch Instructions

Follow these steps to manually apply the patch:


### **1. Modify `include/franka/robot.h`**
1. Open `include/franka/robot.h`.
2. Locate the `Robot` constructor declaration:
   ```cpp
   explicit Robot(const std::string& franka_address,
                  RealtimeConfig realtime_config = RealtimeConfig::kEnforce,
                  size_t log_size = 50);
   ```
3. Change the default value of `realtime_config` from `RealtimeConfig::kEnforce` to `RealtimeConfig::kIgnore`:
   ```cpp
   explicit Robot(const std::string& franka_address,
                  RealtimeConfig realtime_config = RealtimeConfig::kIgnore,
                  size_t log_size = 50);
   ```

### **2. Modify `src/control_loop.cpp`**
1. Open `src/control_loop.cpp`.
2. Locate the following code:
   ```cpp
   bool throw_on_error = robot_.realtimeConfig() == RealtimeConfig::kEnforce;
   ```
3. Change `RealtimeConfig::kEnforce` to `RealtimeConfig::kIgnore`:
   ```cpp
   bool throw_on_error = robot_.realtimeConfig() == RealtimeConfig::kIgnore;
   ```
4. Find the following block of code:
   ```cpp
   if (throw_on_error && !hasRealtimeKernel()) {
     throw RealtimeException("libfranka: Running kernel does not have realtime capabilities.");
   }
   ```
5. Comment out this block by adding `//` at the beginning of each line, like this:
   ```cpp
   // if (throw_on_error && !hasRealtimeKernel()) {
   //   throw RealtimeException("libfranka: Running kernel does not have realtime capabilities.");
   // }
   ```

### **3. Modify `src/robot_impl.h`**
1. Open `src/robot_impl.h`.
2. Locate the following constructor definition:
   ```cpp
   explicit Impl(std::unique_ptr<Network> network,
                 size_t log_size,
                 RealtimeConfig realtime_config = RealtimeConfig::kEnforce);
   ```
3. Change the default value of `realtime_config` from `RealtimeConfig::kEnforce` to `RealtimeConfig::kIgnore`:
   ```cpp
   explicit Impl(std::unique_ptr<Network> network,
                 size_t log_size,
                 RealtimeConfig realtime_config = RealtimeConfig::kIgnore);
   ```

### **4. Rebuild the Project**
1. Navigate to your `catkin` workspace:
   ```bash
   cd ~/catkin_ws
   ```
2. Build the `libfranka` package:
   ```bash
   catkin_make --pkg libfranka
   ```
3. Rebuild your entire workspace:
   ```bash
   catkin_make
   ```

## Notes
- Ensure you have the necessary dependencies installed in your system before building `libfranka`.
- This manual modification is meant for developers familiar with the `libfranka` library and the ROS build system.
- If any issues arise, consult the [Libfranka documentation](https://frankaemika.github.io/docs/).

