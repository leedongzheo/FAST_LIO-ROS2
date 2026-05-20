# FAST_LIO-ROS2
Chạy code fast-lio2
cd ~/Documents/anhthu/fast-lio2/src/
git clone https://github.com/Livox-SDK/livox_ros_driver2.git
git clone -b ROS2 https://github.com/hku-mars/FAST_LIO.git
cd /home/crl/Documents/anhthu/fast-lio2/src/FAST_LIO/include
rm -rf ikd-Tree
git clone https://github.com/hku-mars/ikd-Tree.git temp_repo
mv temp_repo/ikd-Tree ./
rm -rf temp_repo
cd ~/Documents/anhthu/fast-lio2/src/livox_ros_driver2
cp package_ROS2.xml package.xml
sed -i 's/||dev_type==LivoxLidarDeviceType::kLivoxLidarTypeMid360s//g' ~/Documents/anhthu/fast-lio2/src/livox_ros_driver2/src/comm/pub_handler.cpp
cd ~/Documents/anhthu/fast-lio2
colcon build --packages-select livox_ros_driver2 --cmake-args -DROS_EDITION=ROS2 -DDISTRO_ROS=jazzy
source install/setup.zsh
Với Ubuntu Ubuntu 24.04 được viết bằng C17:
Vào file ~/Documents/anhthu/fast-lio2/src/FAST_LIO/CMakeLists.txt
--------------------------------------------------
cmake_minimum_required(VERSION 3.8)
project(fast_lio)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

# Nâng cấp lên chuẩn C++17 cho ROS 2 Jazzy
ADD_COMPILE_OPTIONS(-std=c++17)
set(CMAKE_CXX_FLAGS "-std=c++17 -O3")

add_definitions(-DROOT_DIR=\"${CMAKE_CURRENT_SOURCE_DIR}/\")

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fexceptions")
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
# Xóa bỏ các cờ c++14 và c++0x cũ kỹ gây xung đột
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread -fexceptions")
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

message("Current CPU archtecture: ${CMAKE_SYSTEM_PROCESSOR}")

if(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86)|(X86)|(amd64)|(AMD64)")
  include(ProcessorCount)
  ProcessorCount(N)
  message("Processer number:  ${N}")

  if(N GREATER 4)
    add_definitions(-DMP_EN)
    add_definitions(-DMP_PROC_NUM=3)
    message("core for MP: 3")
  elseif(N GREATER 3)
    add_definitions(-DMP_EN)
    add_definitions(-DMP_PROC_NUM=2)
    message("core for MP: 2")
  else()
    add_definitions(-DMP_PROC_NUM=1)
  endif()
else()
  add_definitions(-DMP_PROC_NUM=1)
endif()

find_package(OpenMP QUIET)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}   ${OpenMP_C_FLAGS}")

find_package(PythonLibs REQUIRED)
find_path(MATPLOTLIB_CPP_INCLUDE_DIRS "matplotlibcpp.h")

# ROS dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_components REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(std_msgs REQUIRED)
find_package(std_srvs REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(pcl_ros REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(livox_ros_driver2 REQUIRED)
find_package(rosidl_default_generators REQUIRED)

set(dependencies
  rclcpp
  rclcpp_components
  geometry_msgs
  nav_msgs
  sensor_msgs
  std_msgs
  std_srvs
  visualization_msgs
  pcl_ros
  pcl_conversions
  livox_ros_driver2
)

# Thirdparty libraries
find_package(Eigen3 REQUIRED)
find_package(PCL REQUIRED COMPONENTS common io)

message(Eigen: ${EIGEN3_INCLUDE_DIR})
message(STATUS "PCL: ${PCL_INCLUDE_DIRS}")

set(msg_files
  "msg/Pose6D.msg"
)

rosidl_generate_interfaces(${PROJECT_NAME}
  ${msg_files}
)
ament_export_dependencies(rosidl_default_runtime)

add_executable(fastlio_mapping src/laserMapping.cpp include/ikd-Tree/ikd_Tree.cpp src/preprocess.cpp)
target_include_directories(fastlio_mapping PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
  ${PCL_INCLUDE_DIRS}
)
target_link_libraries(fastlio_mapping ${PCL_LIBRARIES} ${PYTHON_LIBRARIES} Eigen3::Eigen)
target_include_directories(fastlio_mapping PRIVATE ${PYTHON_INCLUDE_DIRS})

list(APPEND EOL_LIST "foxy" "galactic" "eloquent" "dashing" "crystal")

if($ENV{ROS_DISTRO} IN_LIST EOL_LIST)
  # Custommsg to support foxy & galactic
  rosidl_target_interfaces(fastlio_mapping
    ${PROJECT_NAME} "rosidl_typesupport_cpp")
else()
  rosidl_get_typesupport_target(cpp_typesupport_target
    ${PROJECT_NAME} "rosidl_typesupport_cpp")
  target_link_libraries(fastlio_mapping ${cpp_typesupport_target})
endif()

ament_target_dependencies(fastlio_mapping ${dependencies})

# ---------------- Install --------------- #
install(TARGETS fastlio_mapping
  DESTINATION lib/${PROJECT_NAME}
)

install(
  DIRECTORY config launch rviz
  DESTINATION share/${PROJECT_NAME}
)

ament_package()
------------------------------------------------------------------
cd ~/Documents/anhthu/fast-lio2
colcon build --packages-select fast_lio
source install/setup.zsh
------------------------------------------------------------------
Thêm file nano /home/crl/Documents/anhthu/fast-lio2/src/FAST_LIO/config/city02.yaml
Nội dung file city02.yaml
-------------------------------------------------------------------------------
/**:
  ros__parameters:
    common:
      lid_topic:  "/ouster/points"
      imu_topic:  "/imu/data"
      time_sync_en: false
      time_offset_lidar_to_imu: 0.0

    preprocess:
      lidar_type: 2                # 3 tương ứng với cảm biến Ouster
      scan_line: 128               # Số tia của Ouster trong dataset này
      timestamp_unit: 0            # 0 = Giây (seconds)
      blind: 0.0

    mapping:
      acc_cov: 0.0111974126
      gyr_cov: 0.0102709048
      b_acc_cov: 0.0001175176
      b_gyr_cov: 0.0000913553
      fov_degree: 180.0
      det_range: 100.0
      extrinsic_est_en: true
      
      # Tọa độ tịnh tiến của Ouster (từ mảng T gốc)
      extrinsic_T: [ 0.215, 0.0, 0.018 ]
      
      # Quaternion [1, 0, 0, 0] của Ouster quy đổi ra Ma trận xoay 3x3 (Identity Matrix)
      extrinsic_R: [ 1.0, 0.0, 0.0,
                     0.0, 1.0, 0.0,
                     0.0, 0.0, 1.0 ]

    publish:
      path_en: true
      scan_publish_en: true
      dense_publish_en: true
      scan_bodyframe_pub_en: true

    pcd_save:
      pcd_save_en: true
      interval: -1

    uncertainty:
      point_cov_max: 0.00125
      point_cov_min: 0.00075
      plane_cov_max: 1.0
      plane_cov_min: 0.8
      localize_cov_max: 2.0
      localize_cov_min: 0.3
      localize_thresh_max: 0.7
      localize_thresh_min: 0.2
      cov_threshold: 0.5
-------------------------------------------------------------------------------
colcon build --packages-select fast_lio
source install/setup.zsh
terminal 1:
ros2 launch fast_lio mapping.launch.py config_file:=city02.yaml
KQ: [fastlio_mapping-1] [INFO] [1779206737.041916823] [laser_mapping]: p_pre->lidar_type 3
Cách chạy file city02
Bước 1: Chuyển bag ros1 sang bag ros2 bằng cách:
cd /media/crl/3D/anhthu/City02/sensor_data
ls
KQ:
City02_Bag       data_stamp.csv   Livox_avia  ouster            pack_to_ros2_bag.py
City02_ROS2_Bag  Groundtruth.txt  Livox_tele  ouster_stamp.csv  xsens_imu.csv
nano pack_to_ros2_bag.py
File code pack_to_ros2_bag.py
-------------------------------------------------------------------------------
import os
import csv
import numpy as np
import rclpy
from rclpy.serialization import serialize_message
from rosbag2_py import SequentialWriter, StorageOptions, ConverterOptions, TopicMetadata
from sensor_msgs.msg import PointCloud2, PointField, Imu
from builtin_interfaces.msg import Time
from std_msgs.msg import Header

def ns_to_ros_time(timestamp_ns):
    time_msg = Time()
    time_msg.sec = int(timestamp_ns // 1_000_000_000)
    time_msg.nanosec = int(timestamp_ns % 1_000_000_000)
    return time_msg

def create_velodyne_pointcloud2_msg(timestamp_ns, points_4_cols):
    """Tạo PointCloud2 chuẩn 6 trường của Velodyne: x, y, z, intensity, ring, time"""
    msg = PointCloud2()
    msg.header = Header()
    msg.header.stamp = ns_to_ros_time(timestamp_ns)
    msg.header.frame_id = "camera_init" # Bắt buộc theo chuẩn LIO

    msg.height = 1
    num_points = len(points_4_cols)
    msg.width = num_points
    
    msg.fields = [
        PointField(name='x', offset=0, datatype=PointField.FLOAT32, count=1),
        PointField(name='y', offset=4, datatype=PointField.FLOAT32, count=1),
        PointField(name='z', offset=8, datatype=PointField.FLOAT32, count=1),
        PointField(name='intensity', offset=12, datatype=PointField.FLOAT32, count=1),
        PointField(name='ring', offset=16, datatype=PointField.UINT16, count=1),
        PointField(name='time', offset=18, datatype=PointField.FLOAT32, count=1),
    ]
    
    # Kích thước mỗi điểm là 22 bytes: 4x4 (float32) + 2 (uint16) + 4 (float32)
    # Tuy nhiên, ROS thường căn chỉnh bộ nhớ (padding) thành bội số của 4, nên ta dùng 32 bytes
    msg.point_step = 32 
    msg.row_step = msg.point_step * msg.width
    msg.is_dense = True
    msg.is_bigendian = False

    # Khởi tạo mảng bộ nhớ đệm (buffer) toàn số 0
    buffer = np.zeros(num_points * (msg.point_step // 4), dtype=np.float32)
    
    # Đổ 4 cột x, y, z, intensity vào các vị trí offset
    buffer[0::8] = points_4_cols[:, 0] # x
    buffer[1::8] = points_4_cols[:, 1] # y
    buffer[2::8] = points_4_cols[:, 2] # z
    buffer[3::8] = points_4_cols[:, 3] # intensity
    
    # Trường Time (float32): Tính bằng khoảng cách giữa các điểm giả lập (để bù trừ chuyển động)
    time_array = np.linspace(0.0, 0.1, num_points, dtype=np.float32)
    buffer[4::8] = time_array
    
    # Chuyển đổi buffer thành chuỗi byte thô
    raw_bytes = bytearray(buffer.tobytes())
    
    # Trường Ring (uint16): Chèn vào offset 16 của mỗi điểm (chiếm 2 byte)
    # Ouster có 128 tia, ta tạo mảng tia lặp lại từ 0 đến 127
    ring_array = np.tile(np.arange(128, dtype=np.uint16), int(np.ceil(num_points / 128)))[:num_points]
    
    # Ghi đè byte của Ring vào đúng vị trí offset 16
    for i in range(num_points):
        ring_bytes = ring_array[i].tobytes()
        base_idx = i * msg.point_step
        raw_bytes[base_idx + 16 : base_idx + 18] = ring_bytes

    msg.data = bytes(raw_bytes)
    return msg

def main():
    output_bag_path = 'City02_ROS2_Bag'
    if os.path.exists(output_bag_path):
        print(f"Cảnh báo: Thư mục {output_bag_path} đã tồn tại. Hãy xóa nó!")
        return

    rclpy.init()
    writer = SequentialWriter()
    storage_options = StorageOptions(uri=output_bag_path, storage_id='sqlite3')
    converter_options = ConverterOptions(
        input_serialization_format='cdr',
        output_serialization_format='cdr'
    )
    writer.open(storage_options, converter_options)

    imu_topic = '/imu/data'
    lidar_topic = '/ouster/points'

    writer.create_topic(TopicMetadata(id=1, name=imu_topic, type='sensor_msgs/msg/Imu', serialization_format='cdr'))
    writer.create_topic(TopicMetadata(id=2, name=lidar_topic, type='sensor_msgs/msg/PointCloud2', serialization_format='cdr'))

    print("Đóng gói IMU...")
    with open('xsens_imu.csv', 'r') as f:
        reader = csv.reader(f)
        next(reader) 
        for row in reader:
            if len(row) < 7: continue
            timestamp_ns = int(row[0].strip())
            imu_msg = Imu()
            imu_msg.header.stamp = ns_to_ros_time(timestamp_ns)
            imu_msg.header.frame_id = "camera_init" # Cùng quy chiếu với LiDAR
            
            imu_msg.linear_acceleration.x = float(row[1])
            imu_msg.linear_acceleration.y = float(row[2])
            imu_msg.linear_acceleration.z = float(row[3])
            imu_msg.angular_velocity.x = float(row[4])
            imu_msg.angular_velocity.y = float(row[5])
            imu_msg.angular_velocity.z = float(row[6])
            
            writer.write(imu_topic, serialize_message(imu_msg), timestamp_ns)

    print("Đóng gói LiDAR chuẩn Velodyne...")
    with open('ouster_stamp.csv', 'r') as f:
        reader = csv.reader(f)
        next(reader) 
        count = 0
        for row in reader:
            timestamp_ns = int(row[0].strip())
            bin_filename = f"{row[0].strip()}.bin" 
            bin_path = os.path.join('ouster', bin_filename)
            
            # if os.path.exists(bin_path):
            #     raw_data = np.fromfile(bin_path, dtype=np.float32)
            #     clean_data = raw_data[256:]
            #     num_points = len(clean_data) // 9
            #     data_9_cols = clean_data[:num_points * 9].reshape(-1, 9)
            #     points_4_cols = data_9_cols[:, :4].astype(np.float32)
                
            #     # Dùng hàm mới!
            #     pc_msg = create_velodyne_pointcloud2_msg(timestamp_ns, points_4_cols)
            #     writer.write(lidar_topic, serialize_message(pc_msg), timestamp_ns)
            # if os.path.exists(bin_path):
            #     # Đọc thẳng dữ liệu, KHÔNG cắt bỏ 256
            #     raw_data = np.fromfile(bin_path, dtype=np.float32)
                
            #     num_points = len(raw_data) // 9
            #     data_9_cols = raw_data[:num_points * 9].reshape(-1, 9)
                
            #     # Lấy 4 cột chuẩn: X, Y, Z, Intensity
            #     points_4_cols = data_9_cols[:, :4].astype(np.float32)
                
            #     # Gọi hàm giả lập 6 trường Velodyne (đã viết ở bước trước)
            #     pc_msg = create_velodyne_pointcloud2_msg(timestamp_ns, points_4_cols)
            #     writer.write(lidar_topic, serialize_message(pc_msg), timestamp_ns) 
            # if os.path.exists(bin_path):
            #     # 1. Đọc thẳng dữ liệu, không cắt 256
            #     raw_data = np.fromfile(bin_path, dtype=np.float32)
                
            #     # 2. Gom theo khối 8 float
            #     num_points = len(raw_data) // 8
            #     data_8_cols = raw_data[:num_points * 8].reshape(-1, 8)
                
            #     # 3. Lấy 4 cột đầu tiên: X, Y, Z, Intensity
            #     points_4_cols = data_8_cols[:, :4].astype(np.float32)
                
            #     # --- THÊM LỚP KHIÊN BẢO VỆ Ở ĐÂY ---
            #     # Bước 3.1: Loại bỏ NGAY các dòng chứa NaN hoặc Inf
            #     is_valid = np.isfinite(points_4_cols).all(axis=1)
            #     safe_points = points_4_cols[is_valid]
                
            #     # Bước 3.2: Lọc khoảng cách (chỉ tính trên những điểm an toàn)
            #     # Tính bình phương khoảng cách (X^2 + Y^2 + Z^2)
            #     ranges_sq = np.sum(safe_points[:, :3]**2, axis=1)
            #     valid_mask = (
            #         (np.abs(points_4_cols[:, 0]) < 150.0) &  # X nhỏ hơn 150m
            #         (np.abs(points_4_cols[:, 1]) < 150.0) &  # Y nhỏ hơn 150m
            #         (np.abs(points_4_cols[:, 2]) < 150.0) &  # Z nhỏ hơn 150m
            #         (np.abs(points_4_cols[:, 0]) > 0.1)      # Loại bỏ điểm mù (bám dính vào xe)
            #     )
            #     # Bước 3.3: Lấy những điểm cách xa gốc trên 0.1m (bình phương > 0.01)
            #     # valid_points = safe_points[ranges_sq > 0.01]
            #     valid_points = points_4_cols[valid_mask]

            #     # Gọi hàm tạo Velodyne 
            #     pc_msg = create_velodyne_pointcloud2_msg(timestamp_ns, valid_points)
            #     writer.write(lidar_topic, serialize_message(pc_msg), timestamp_ns) 
            if os.path.exists(bin_path):
                # 1. Đọc thẳng dữ liệu thô
                raw_data = np.fromfile(bin_path, dtype=np.float32)
                
                # 2. CHÌA KHÓA VÀNG: Gom theo khối 12 float (48 bytes) CHỨ KHÔNG PHẢI 8!
                num_points = len(raw_data) // 12
                data_12_cols = raw_data[:num_points * 12].reshape(-1, 12)
                
                # 3. Tạo mảng 4 cột an toàn
                points_4_cols = np.zeros((num_points, 4), dtype=np.float32)
                
                # Lấy ĐÚNG 3 cột đầu tiên (X, Y, Z) - Nơi chứa tọa độ thật
                points_4_cols[:, :3] = data_12_cols[:, :3]
                
                # Cột thứ 4 (Intensity) trong file bin đang bị lỗi chứa số -inf
                # Ta ép tất cả cường độ bằng 100.0 để Fast-LIO2 không bị hoảng loạn
                points_4_cols[:, 3] = 100.0 
                
                # 4. Lớp khiên bảo vệ (Chỉ lấy điểm từ 0.1m đến 150m)
                ranges_sq = np.sum(points_4_cols[:, :3]**2, axis=1)
                # 0.01 là bình phương của 0.1m, 22500 là bình phương của 150m
                valid_mask = (ranges_sq > 0.01) & (ranges_sq < 22500.0)
                valid_points = points_4_cols[valid_mask]

                # 5. Gọi hàm tạo Velodyne 
                pc_msg = create_velodyne_pointcloud2_msg(timestamp_ns, valid_points)
                writer.write(lidar_topic, serialize_message(pc_msg), timestamp_ns)
                count += 1
                if count % 100 == 0:
                    print(f"Đã xử lý {count} khung hình LiDAR...")

    print(f"Xong! File lưu tại: {output_bag_path}")
    rclpy.shutdown()

if __name__ == '__main__':
    main()
-------------------------------------------------------------------------------
python3 pack_to_ros2_bag.py
Kiểm tra thông tin dữ liệu:
ros2 bag info City02_ROS2_Bag
KQ:
Files:             City02_ROS2_Bag_0.db3
Bag size:          16.8 GiB
Storage id:        sqlite3
ROS Distro:        jazzy
Duration:          624.756752968s
Start:             Dec 21 2022 08:13:07.156928062 (1671631987.156928062)
End:               Dec 21 2022 08:23:31.913681030 (1671632611.913681030)
Messages:          68723
Topic information: Topic: /imu/data | Type: sensor_msgs/msg/Imu | Count: 62477 | Serialization Format: cdr
                   Topic: /ouster/points | Type: sensor_msgs/msg/PointCloud2 | Count: 6246 | Serialization Format: cdr
Service:           0
Service information:
Danh sách 5 file đầu tiên
ls ouster | head -n 5
KQ:
1671631987059262720.bin
1671631987159277056.bin
1671631987259305216.bin
1671631987359341568.bin
1671631987459377408.bin
Kiem tra thông tin file:
ls -lh /media/crl/3D/anhthu/City02/sensor_data/City02_ROS2_Bag/City02_ROS2_Bag_0.db3
KQ:
----------------------------------------------------------------------------
-rw-r--r-- 1 crl crl 17G May 19 10:13 /media/crl/3D/anhthu/City02/sensor_data/City02_ROS2_Bag/City02_ROS2_Bag_0.db3
----------------------------------------------------------------------------
terminal 2:
ros2 bag play /media/crl23:12 19/05/2026/3D/anhthu/City02/sensor_data/City02_ROS2_Bag --clock
