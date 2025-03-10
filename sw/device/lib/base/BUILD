# Copyright lowRISC contributors.
# Licensed under the Apache License, Version 2.0, see LICENSE for details.
# SPDX-License-Identifier: Apache-2.0

package(default_visibility = ["//visibility:public"])

load("//rules:cross_platform.bzl", "dual_cc_library", "dual_inputs")
load("//rules:opentitan.bzl", "OPENTITAN_CPU")

cc_library(
    name = "global_mock",
    hdrs = ["global_mock.h"],
)

cc_library(
    name = "stdasm",
    hdrs = ["stdasm.h"],
)

cc_library(
    name = "macros",
    hdrs = ["macros.h"],
)

cc_library(
    name = "math_builtins",
    srcs = ["math_builtins.c"],

    # This library defines polyfills, so we need to ensure LLVM doesn't make
    # the polyfills call themselves.
    copts = ["-fno-builtin"],
    visibility = ["//visibility:private"],
    deps = [":macros"],
)

cc_test(
    name = "math_builtins_unittest",
    srcs = ["math_builtins_unittest.cc"],
    deps = [
        ":bitfield",
        ":math_builtins",
        "@googletest//:gtest_main",
    ],
)

cc_library(
    name = "math",
    srcs = ["math.c"],
    hdrs = ["math.h"],
    deps = [":math_builtins"],
)

cc_test(
    name = "math_unittest",
    srcs = ["math_unittest.cc"],
    deps = [
        ":math",
        "@googletest//:gtest_main",
    ],
)

cc_library(
    name = "bitfield",
    srcs = ["bitfield.c"],
    hdrs = ["bitfield.h"],
    deps = [
        ":macros",
        ":math",
    ],
)

cc_library(
    name = "memory",
    srcs = ["memory.c"],
    hdrs = ["memory.h"],

    # This library defines memcpy(), so we can't have LLVM rewriting memcpy
    # into a call to itself.
    copts = ["-fno-builtin"],
    deps = [
        ":macros",
    ],
)

cc_library(
    name = "hardened",
    srcs = ["hardened.c"],
    hdrs = ["hardened.h"],
    deps = [
        ":macros",
        ":memory",
        ":stdasm",
    ],
)

cc_test(
    name = "hardened_unittest",
    srcs = ["hardened_unittest.cc"],
    defines = [
        "OT_OFF_TARGET_TEST",
    ],
    deps = [
        ":hardened",
        "@googletest//:gtest_main",
    ],
)

cc_library(
    name = "random_order",
    srcs = ["random_order.c"],
    hdrs = ["random_order.h"],
    deps = [":bitfield"],
)

cc_library(
    name = "hardened_memory",
    srcs = ["hardened_memory.c"],
    hdrs = ["hardened_memory.h"],
    deps = [
        ":hardened",
        ":macros",
        ":memory",
        ":random_order",
    ],
)

cc_test(
    name = "hardened_memory_unittest",
    srcs = ["hardened_memory_unittest.cc"],
    deps = [
        ":hardened_memory",
        ":random_order",
        "@googletest//:gtest_main",
    ],
)

dual_cc_library(
    name = "csr",
    srcs = dual_inputs(
        host = ["//sw/device/silicon_creator/lib/base:mock_csr.cc"],
    ),
    hdrs = dual_inputs(
        host = [
            "//sw/device/silicon_creator/lib/base:mock_csr.h",
        ],
        shared = [
            "csr.h",
            "csr_registers.h",
        ],
    ),
    deps = dual_inputs(
        host = [
            "global_mock",
            "@googletest//:gtest",
        ],
        shared = [
            ":macros",
            ":stdasm",
        ],
    ),
)

cc_test(
    name = "mock_csr_unittest",
    srcs = ["//sw/device/silicon_creator/lib/base:mock_csr_test.cc"],
    deps = [
        ":csr",
        "@googletest//:gtest_main",
    ],
)

cc_library(
    name = "multibits",
    hdrs = [
        "multibits.h",
        "multibits_asm.h",
    ],
)

dual_cc_library(
    name = "mmio",
    srcs = dual_inputs(
        host = ["mock_mmio.cc"],
        # NOTE: mmio.c is shared because it provides mmio_memcpy and friends.
        shared = ["mmio.c"],
    ),
    hdrs = dual_inputs(
        host = [
            "mock_mmio.h",
            "mock_mmio_test_utils.h",
        ],
        shared = ["mmio.h"],
    ),
    deps = dual_inputs(
        host = [
            "@googletest//:gtest",
        ],
        shared = [
            ":bitfield",
            ":macros",
            ":memory",
        ],
    ),
)

cc_test(
    name = "mmio_unittest",
    srcs = ["mock_mmio_test.cc"],
    deps = [
        ":mmio",
        "@googletest//:gtest_main",
    ],
)

dual_cc_library(
    name = "abs_mmio",
    srcs = dual_inputs(
        device = ["abs_mmio.c"],
        host = ["mock_abs_mmio.cc"],
    ),
    hdrs = dual_inputs(
        host = [
            "mock_abs_mmio.h",
        ],
        shared = ["abs_mmio.h"],
    ),
    deps = dual_inputs(
        host = [
            "global_mock",
            "@googletest//:gtest",
        ],
        shared = [
            ":mmio",
            ":macros",
        ],
    ),
)
