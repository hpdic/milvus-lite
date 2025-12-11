# Copyright (C) 2019-2024 Zilliz. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License"); you may not
# use this file except in compliance with the License. You may obtain a copy of
# the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
# License for the specific language governing permissions and limitations under
# the License

# --- 开始补丁 ---
find_package(Protobuf REQUIRED)

if(NOT TARGET protobuf::protobuf)
    if(TARGET protobuf::libprotobuf)
        add_library(protobuf::protobuf ALIAS protobuf::libprotobuf)
    else()
        # 最后的兜底：如果连 target 都没有，就用变量手动创建
        add_library(protobuf::protobuf UNKNOWN IMPORTED)
        set_target_properties(protobuf::protobuf PROPERTIES
            IMPORTED_LOCATION "${Protobuf_LIBRARY}"
            INTERFACE_INCLUDE_DIRECTORIES "${Protobuf_INCLUDE_DIR}"
        )
    endif()
endif()
# --- 结束补丁 ---

# --- ARROW PATCH START ---
find_package(Arrow REQUIRED)

if(NOT TARGET arrow::arrow)
    # 尝试找标准名 Arrow::arrow_shared
    if(TARGET Arrow::arrow_shared)
        add_library(arrow::arrow ALIAS Arrow::arrow_shared)
    # 尝试找旧版名字 arrow_shared
    elseif(TARGET arrow_shared)
        add_library(arrow::arrow ALIAS arrow_shared)
    else()
        # 最后的保底：手动指向文件
        add_library(arrow::arrow UNKNOWN IMPORTED)
        # 注意：这里假设 FindArrow 模块填充了 ARROW_SHARED_LIB 变量
        # 如果报错，可以尝试把下一行改成具体路径 /usr/local/lib/libarrow.so
        set_target_properties(arrow::arrow PROPERTIES
            IMPORTED_LOCATION "${ARROW_SHARED_LIB}"
            INTERFACE_INCLUDE_DIRECTORIES "${ARROW_INCLUDE_DIR}"
        )
    endif()
endif()
# --- ARROW PATCH END ---

include_directories(${CMAKE_CURRENT_LIST_DIR})
add_library(milite ${LITE_BUILD_TYPE}
  ${CMAKE_CURRENT_LIST_DIR}/storage/collection_meta.cpp
  ${CMAKE_CURRENT_LIST_DIR}/storage/collection_data.cpp
  ${CMAKE_CURRENT_LIST_DIR}/storage/bm25_stats.cpp  
  ${CMAKE_CURRENT_LIST_DIR}/storage/storage.cpp
  ${CMAKE_CURRENT_LIST_DIR}/timer.cpp
  ${CMAKE_CURRENT_LIST_DIR}/index.cpp
  ${CMAKE_CURRENT_LIST_DIR}/milvus_local.cpp
  ${CMAKE_CURRENT_LIST_DIR}/segcore_wrapper.cpp
  ${CMAKE_CURRENT_LIST_DIR}/milvus_proxy.cpp
  ${CMAKE_CURRENT_LIST_DIR}/create_collection_task.cpp
  ${CMAKE_CURRENT_LIST_DIR}/create_index_task.cpp
  ${CMAKE_CURRENT_LIST_DIR}/search_task.cpp
  ${CMAKE_CURRENT_LIST_DIR}/insert_task.cpp
  ${CMAKE_CURRENT_LIST_DIR}/query_task.cpp
  ${CMAKE_CURRENT_LIST_DIR}/delete_task.cpp
  ${CMAKE_CURRENT_LIST_DIR}/upsert_task.cpp  
  ${CMAKE_CURRENT_LIST_DIR}/schema_util.cpp
  ${CMAKE_CURRENT_LIST_DIR}/re_scorer.cpp
  ${CMAKE_CURRENT_LIST_DIR}/hybrid_search_task.cpp
  ${CMAKE_CURRENT_LIST_DIR}/function/function_executor.cpp
  ${CMAKE_CURRENT_LIST_DIR}/function/bm25_function.cpp
)

target_link_libraries(
  milite
  PUBLIC
  parser
  milvus_core
  milvus_log
  yaml-cpp
  roaring::roaring
  SQLiteCpp
  ${antlr4-cppruntime_LIBRARIES}
  ${MARISA_LIBRARIES}
  TBB::tbb
  ${PROTOBUF_LIBRARIES}
  ${ARROW_LIBRARIES}
  simdjson
  tantivy_binding
)

add_library( 
  milvus_service
  STATIC
  "${CMAKE_SOURCE_DIR}/src/milvus_service_impl.cpp"
)

target_link_libraries(
  milvus_service
  PUBLIC
  milvus_grpc_service
)

add_executable(milvus server.cpp)

target_link_libraries(
  milvus
  milvus_service
  milite
)

if(ENABLE_UNIT_TESTS)
  add_subdirectory(unittest)
endif()
