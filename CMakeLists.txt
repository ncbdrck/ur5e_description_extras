cmake_minimum_required(VERSION 3.0.2)
project(ur5e_description_extras)

find_package(catkin REQUIRED)

catkin_package()

install(
  DIRECTORY config launch models urdf
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
