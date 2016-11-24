---
layout: post
title:  "Android系统源码目录结构"
date:   2016-11-24 1:05:00
catalog:  true
tags:

   - Android
   - 源码目录
---



## Android源码目录结构

### 一级目录

Android 4.0

|-- Makefile

|-- bionic （bionic C库）

|-- bootable （启动引导相关代码）

|-- build （存放系统编译规则及generic等基础开发包配置）

|-- cts （Android兼容性测试套件标准）

|-- dalvik （dalvik JAVA虚拟机）

|-- development （应用程序开发相关）

|-- external （android使用的一些开源的模组）

|-- frameworks （核心框架——java及C++语言）

|-- hardware （部分厂家开源的硬解适配层HAL代码）

|-- out （编译完成后的代码输出与此目录）

|-- packages （应用程序包）

|-- prebuilt （x86和arm架构下预编译的一些资源）

|-- sdk （sdk及模拟器）

|-- system （底层文件系统库、应用及组件——C语言）

`-- vendor （厂商定制代码）

### 二级目录

#### bootable 目录

|-- bootloader （适合各种bootloader的通用代码）

| `-- legacy （估计不能直接使用，可以参考）

| |-- arch_armv6 （V6架构，几个简单的汇编文件）

| |-- arch_msm7k （高通7k处理器架构的几个基本驱动）

| |-- include （通用头文件和高通7k架构头文件）

| |-- libboot （启动库，都写得很简单）

| |-- libc （一些常用的c函数）

| |-- nandwrite （nandwirte函数实现）

| `-- usbloader （usbloader实现）

|-- diskinstaller （android镜像打包器，x86可生产iso）

`-- recovery （系统恢复相关）

|-- edify （升级脚本使用的edify脚本语言）

|-- etc （init.rc恢复脚本）

|-- minui （一个简单的UI）

|-- minzip （一个简单的压缩工具）

|-- mtdutils （mtd工具）

|-- res （资源）

| `-- images （一些图片）

|-- tools （工具）

| `-- ota （OTA Over The Air Updates升级工具）

`-- updater （升级器）