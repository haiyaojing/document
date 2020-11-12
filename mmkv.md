## MMKV——基于 mmap 的高性能通用 key-value 组件
项目原始git[链接地址](https://github.com/Tencent/MMKV)

基于git导出unity可使用的接口git[链接地址](https://github.com/haiyaojing/MMKV/blob/master/readme_cn.md)

单独接入android、iOS、或者win32可参考原始git的说明书，下面我给出我们项目导出给unity静态库的方式。

1. Win32
   使用vs打开Win32/Win32.sln,编译即可
2. Android
   使用AS打开Android/MMKV，编译即可
3. IOS
   使用xcode打开iOS/MMKV/MMKV.xcodeproj，编译即可
   

其中，Android&Win32使用的是Core/native-bridge.h进行导出，本质上还是对原先库的调用

```c++
//
// Created by Hai Yaojing on 2020/10/28.
//

#ifndef MMKV_NATIVE_BRIDGE_H
#define MMKV_NATIVE_BRIDGE_H

#include "MMKVPredef.h"

#include "MMBuffer.h"
#include "MMKV.h"
#include "MMKVLog.h"
#include "MemoryFile.h"
#include <string>
#include <iostream>

using namespace std;
using namespace mmkv;

#ifdef MMKV_ANDROID
#define EXPORT_DLL_API __attribute__((visibility("default")))
#else
#define EXPORT_DLL_API __declspec(dllexport)
#endif


extern MMKV *defaultInstance();
extern MMKV *getInstance(const char *mapId);
extern "C" EXPORT_DLL_API void __init(const char * path, int logLevel);
extern "C" EXPORT_DLL_API void __encodeString(const char *mapId, const char *key, const char *value);
extern "C" EXPORT_DLL_API const char* __decodeString(const char *mapId, const char *key);
extern "C" EXPORT_DLL_API void __encodeBool(const char *mapId, const char *key, const bool value);
extern "C" EXPORT_DLL_API bool __decodeBool(const char *mapId, const char *key, const bool defaultValue);
extern "C" EXPORT_DLL_API void __encodeInt(const char *mapId, const char *key, const int32_t value);
extern "C" EXPORT_DLL_API int32_t __decodeInt(const char *mapId, const char *key, const int32_t defaultValue);
extern "C" EXPORT_DLL_API void __encodeLong(const char *mapId, const char *key, const int64_t value);
extern "C" EXPORT_DLL_API int64_t __decodeLong(const char *mapId, const char *key, const int64_t defaultValue);
extern "C" EXPORT_DLL_API void __encodeFloat(const char *mapId, const char *key, const float value);
extern "C" EXPORT_DLL_API float __decodeFloat(const char *mapId, const char *key, const float defaultValue);
extern "C" EXPORT_DLL_API void __encodeDouble(const char *mapId, const char *key, const double value);
extern "C" EXPORT_DLL_API double __decodeDouble(const char *mapId, const char *key, const double defaultValue);
extern "C" EXPORT_DLL_API size_t __totalSize(const char *mapId);
extern "C" EXPORT_DLL_API size_t __actualSize(const char *mapId);
extern "C" EXPORT_DLL_API void __removeValueForKey(const char *mapId, const char *key);
extern "C" EXPORT_DLL_API void __removeValuesForKeys(const char *mapId, const char **keys, const int size);
extern "C" EXPORT_DLL_API bool __containsKey(const char *mapId, const char *key);
extern "C" EXPORT_DLL_API const char* __getAllKey(const char *mapId);
extern "C" EXPORT_DLL_API void __sync(const char *mapId);
extern "C" EXPORT_DLL_API void __async(const char *mapId);
extern "C" EXPORT_DLL_API void __clearAll(const char *mapId);
extern "C" EXPORT_DLL_API void __close(const char *mapId);
extern "C" EXPORT_DLL_API void __freeNewMemory(char *pBuffer);

#endif //MMKV_NATIVE_BRIDGE_H

```

iOS的导出代码是在原先的libmmkv.mm的后方加入了导出方法，见下方代码中的extern "C"部分

```objective-c
/*
 * Tencent is pleased to support the open source community by making
 * MMKV available.
 *
 * Copyright (C) 2020 THL A29 Limited, a Tencent company.
 * All rights reserved.
 *
 * Licensed under the BSD 3-Clause License (the "License"); you may not use
 * this file except in compliance with the License. You may obtain a copy of
 * the License at
 *
 *       https://opensource.org/licenses/BSD-3-Clause
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

#import "MMKV.h"
#import <MMKVCore/MMKV.h>
#import <MMKVCore/MMKVLog.h>
#import <MMKVCore/ScopedLock.hpp>
#import <MMKVCore/ThreadLock.h>
#import <MMKVCore/openssl_md5.h>

#if defined(MMKV_IOS) && !defined(MMKV_IOS_EXTENSION)
#import <UIKit/UIKit.h>
#endif

#if __has_feature(objc_arc)
#error This file must be compiled with MRC. Use -fno-objc-arc flag.
#endif

using namespace std;

static NSMutableDictionary *g_instanceDic = nil;
static mmkv::ThreadLock *g_lock;
static id<MMKVHandler> g_callbackHandler = nil;
static bool g_isLogRedirecting = false;
static NSString *g_basePath = nil;
static NSString *g_groupPath = nil;

#if defined(MMKV_IOS) && !defined(MMKV_IOS_EXTENSION)
static BOOL g_isRunningInAppExtension = NO;
#endif

static void LogHandler(mmkv::MMKVLogLevel level, const char *file, int line, const char *function, NSString *message);
static mmkv::MMKVRecoverStrategic ErrorHandler(const string &mmapID, mmkv::MMKVErrorType errorType);
static void ContentChangeHandler(const string &mmapID);

@implementation MMKV {
    NSString *m_mmapID;
    NSString *m_mmapKey;
    mmkv::MMKV *m_mmkv;
    uint64_t m_lastAccessTime;
}

#pragma mark - init

// protect from some old code that don't call +initializeMMKV:
+ (void)initialize {
    if (self == MMKV.class) {
        g_instanceDic = [[NSMutableDictionary alloc] init];
        g_lock = new mmkv::ThreadLock();
        g_lock->initialize();

        mmkv::MMKV::minimalInit([self mmkvBasePath].UTF8String);

#if defined(MMKV_IOS) && !defined(MMKV_IOS_EXTENSION)
        // just in case someone forget to set the MMKV_IOS_EXTENSION macro
        if ([[[NSBundle mainBundle] bundlePath] hasSuffix:@".appex"]) {
            g_isRunningInAppExtension = YES;
        }
        if (!g_isRunningInAppExtension) {
            auto appState = [UIApplication sharedApplication].applicationState;
            auto isInBackground = (appState == UIApplicationStateBackground);
            mmkv::MMKV::setIsInBackground(isInBackground);
            MMKVInfo("appState:%ld", (long) appState);

            [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(didEnterBackground) name:UIApplicationDidEnterBackgroundNotification object:nil];
            [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(didBecomeActive) name:UIApplicationDidBecomeActiveNotification object:nil];
        }
#endif
    }
}

+ (NSString *)initializeMMKV:(nullable NSString *)rootDir {
    return [MMKV initializeMMKV:rootDir logLevel:MMKVLogInfo];
}

static BOOL g_hasCalledInitializeMMKV = NO;

+ (NSString *)initializeMMKV:(nullable NSString *)rootDir logLevel:(MMKVLogLevel)logLevel {
    if (g_hasCalledInitializeMMKV) {
        MMKVWarning("already called +initializeMMKV before, ignore this request");
        return [self mmkvBasePath];
    }
    g_hasCalledInitializeMMKV = YES;

    g_basePath = (rootDir != nil) ? [rootDir retain] : [self mmkvBasePath];
    mmkv::MMKV::initializeMMKV(g_basePath.UTF8String, (mmkv::MMKVLogLevel) logLevel);

    return [self mmkvBasePath];
}

+ (NSString *)initializeMMKV:(nullable NSString *)rootDir groupDir:(NSString *)groupDir logLevel:(MMKVLogLevel)logLevel {
    auto ret = [MMKV initializeMMKV:rootDir logLevel:logLevel];

    g_groupPath = [[groupDir stringByAppendingPathComponent:@"mmkv"] retain];
    MMKVInfo("groupDir: %@", g_groupPath);

    return ret;
}

+ (instancetype)getInstance:(const char*)mapId {
    if (!mapId) return [MMKV defaultMMKV];
    
    NSString* pMapId = [NSString stringWithUTF8String:mapId];
    if (pMapId.length <= 0) return [MMKV defaultMMKV];
    
    return [MMKV mmkvWithID:pMapId];
}

// a generic purpose instance
+ (instancetype)defaultMMKV {
    return [MMKV mmkvWithID:(@"" DEFAULT_MMAP_ID) cryptKey:nil rootPath:nil mode:MMKVSingleProcess];
}

// any unique ID (com.tencent.xin.pay, etc)
+ (instancetype)mmkvWithID:(NSString *)mmapID {
    return [MMKV mmkvWithID:mmapID cryptKey:nil rootPath:nil mode:MMKVSingleProcess];
}

+ (instancetype)mmkvWithID:(NSString *)mmapID mode:(MMKVMode)mode {
    auto rootPath = (mode == MMKVSingleProcess) ? nil : g_groupPath;
    return [MMKV mmkvWithID:mmapID cryptKey:nil rootPath:rootPath mode:mode];
}

+ (instancetype)mmkvWithID:(NSString *)mmapID cryptKey:(NSData *)cryptKey {
    return [MMKV mmkvWithID:mmapID cryptKey:cryptKey rootPath:nil mode:MMKVSingleProcess];
}

+ (instancetype)mmkvWithID:(NSString *)mmapID cryptKey:(nullable NSData *)cryptKey mode:(MMKVMode)mode {
    auto rootPath = (mode == MMKVSingleProcess) ? nil : g_groupPath;
    return [MMKV mmkvWithID:mmapID cryptKey:cryptKey rootPath:rootPath mode:mode];
}

+ (instancetype)mmkvWithID:(NSString *)mmapID rootPath:(nullable NSString *)rootPath {
    return [MMKV mmkvWithID:mmapID cryptKey:nil rootPath:rootPath mode:MMKVSingleProcess];
}

+ (instancetype)mmkvWithID:(NSString *)mmapID relativePath:(nullable NSString *)relativePath {
    return [MMKV mmkvWithID:mmapID cryptKey:nil rootPath:relativePath mode:MMKVSingleProcess];
}

+ (instancetype)mmkvWithID:(NSString *)mmapID cryptKey:(NSData *)cryptKey rootPath:(nullable NSString *)rootPath {
    return [MMKV mmkvWithID:mmapID cryptKey:cryptKey rootPath:rootPath mode:MMKVSingleProcess];
}

+ (instancetype)mmkvWithID:(NSString *)mmapID cryptKey:(nullable NSData *)cryptKey relativePath:(nullable NSString *)relativePath {
    return [MMKV mmkvWithID:mmapID cryptKey:cryptKey rootPath:relativePath mode:MMKVSingleProcess];
}

// relatePath and MMKVMultiProcess mode can't be set at the same time, so we hide this method from public
+ (instancetype)mmkvWithID:(NSString *)mmapID cryptKey:(NSData *)cryptKey rootPath:(nullable NSString *)rootPath mode:(MMKVMode)mode {
    if (!g_hasCalledInitializeMMKV) {
        MMKVError("MMKV not initialized properly, must call +initializeMMKV: in main thread before calling any other MMKV methods");
    }
    if (mmapID.length <= 0) {
        return nil;
    }

    SCOPED_LOCK(g_lock);

    if (mode == MMKVMultiProcess) {
        if (!rootPath) {
            rootPath = g_groupPath;
        }
        if (!rootPath) {
            MMKVError("Getting a multi-process MMKV [%@] without setting groupDir makes no sense", mmapID);
            MMKV_ASSERT(0);
        }
    }
    NSString *kvKey = [MMKV mmapKeyWithMMapID:mmapID rootPath:rootPath];
    MMKV *kv = [g_instanceDic objectForKey:kvKey];
    if (kv == nil) {
        kv = [[MMKV alloc] initWithMMapID:mmapID cryptKey:cryptKey rootPath:rootPath mode:mode];
        if (!kv->m_mmkv) {
            [kv release];
            return nil;
        }
        kv->m_mmapKey = kvKey;
        [g_instanceDic setObject:kv forKey:kvKey];
        [kv release];
    }
    kv->m_lastAccessTime = [NSDate timeIntervalSinceReferenceDate];
    return kv;
}

- (instancetype)initWithMMapID:(NSString *)mmapID cryptKey:(NSData *)cryptKey rootPath:(NSString *)rootPath mode:(MMKVMode)mode {
    if (self = [super init]) {
        string pathTmp;
        if (rootPath.length > 0) {
            pathTmp = rootPath.UTF8String;
        }
        string cryptKeyTmp;
        if (cryptKey.length > 0) {
            cryptKeyTmp = string((char *) cryptKey.bytes, cryptKey.length);
        }
        string *rootPathPtr = pathTmp.empty() ? nullptr : &pathTmp;
        string *cryptKeyPtr = cryptKeyTmp.empty() ? nullptr : &cryptKeyTmp;
        m_mmkv = mmkv::MMKV::mmkvWithID(mmapID.UTF8String, (mmkv::MMKVMode) mode, cryptKeyPtr, rootPathPtr);
        if (!m_mmkv) {
            return self;
        }
        m_mmapID = [[NSString alloc] initWithUTF8String:m_mmkv->mmapID().c_str()];

#if defined(MMKV_IOS) && !defined(MMKV_IOS_EXTENSION)
        if (!g_isRunningInAppExtension) {
            [[NSNotificationCenter defaultCenter] addObserver:self
                                                     selector:@selector(onMemoryWarning)
                                                         name:UIApplicationDidReceiveMemoryWarningNotification
                                                       object:nil];
        }
#endif
    }
    return self;
}

- (void)dealloc {
    [self clearMemoryCache];

    [[NSNotificationCenter defaultCenter] removeObserver:self];

    MMKVInfo("dealloc %@", m_mmapID);
    [m_mmapID release];

    if (m_mmkv) {
        m_mmkv->close();
        m_mmkv = nullptr;
    }

    [super dealloc];
}

#pragma mark - Application state

#if defined(MMKV_IOS) && !defined(MMKV_IOS_EXTENSION)
- (void)onMemoryWarning {
    MMKVInfo("cleaning on memory warning %@", m_mmapID);

    [self clearMemoryCache];
}

+ (void)didEnterBackground {
    mmkv::MMKV::setIsInBackground(true);
    MMKVInfo("isInBackground:%d", true);
}

+ (void)didBecomeActive {
    mmkv::MMKV::setIsInBackground(false);
    MMKVInfo("isInBackground:%d", false);
}
#endif

- (void)clearAll {
    m_mmkv->clearAll();
}

- (void)clearMemoryCache {
    if (m_mmkv) {
        m_mmkv->clearMemoryCache();
    }
}

- (void)close {
    SCOPED_LOCK(g_lock);
    MMKVInfo("closing %@", m_mmapID);

    [g_instanceDic removeObjectForKey:m_mmapKey];
}

- (void)trim {
    m_mmkv->trim();
}

#pragma mark - encryption & decryption

#ifndef MMKV_DISABLE_CRYPT

- (nullable NSData *)cryptKey {
    auto str = m_mmkv->cryptKey();
    if (str.length() > 0) {
        return [NSData dataWithBytes:str.data() length:str.length()];
    }
    return nil;
}

- (BOOL)reKey:(nullable NSData *)newKey {
    string key;
    if (newKey.length > 0) {
        key = string((char *) newKey.bytes, newKey.length);
    }
    return m_mmkv->reKey(key);
}

- (void)checkReSetCryptKey:(nullable NSData *)cryptKey {
    if (cryptKey.length > 0) {
        string key = string((char *) cryptKey.bytes, cryptKey.length);
        m_mmkv->checkReSetCryptKey(&key);
    } else {
        m_mmkv->checkReSetCryptKey(nullptr);
    }
}

#else

- (nullable NSData *)cryptKey {
    return nil;
}

- (BOOL)reKey:(nullable NSData *)newKey {
    return NO;
}

- (void)checkReSetCryptKey:(nullable NSData *)cryptKey {
}

#endif // MMKV_DISABLE_CRYPT

#pragma mark - set & get

- (BOOL)setObject:(nullable NSObject<NSCoding> *)object forKey:(NSString *)key {
    return m_mmkv->set(object, key);
}

- (BOOL)setBool:(BOOL)value forKey:(NSString *)key {
    return m_mmkv->set((bool) value, key);
}

- (BOOL)setInt32:(int32_t)value forKey:(NSString *)key {
    return m_mmkv->set(value, key);
}

- (BOOL)setUInt32:(uint32_t)value forKey:(NSString *)key {
    return m_mmkv->set(value, key);
}

- (BOOL)setInt64:(int64_t)value forKey:(NSString *)key {
    return m_mmkv->set(value, key);
}

- (BOOL)setUInt64:(uint64_t)value forKey:(NSString *)key {
    return m_mmkv->set(value, key);
}

- (BOOL)setFloat:(float)value forKey:(NSString *)key {
    return m_mmkv->set(value, key);
}

- (BOOL)setDouble:(double)value forKey:(NSString *)key {
    return m_mmkv->set(value, key);
}

- (BOOL)setString:(NSString *)value forKey:(NSString *)key {
    return [self setObject:value forKey:key];
}

- (BOOL)setDate:(NSDate *)value forKey:(NSString *)key {
    return [self setObject:value forKey:key];
}

- (BOOL)setData:(NSData *)value forKey:(NSString *)key {
    return [self setObject:value forKey:key];
}

- (id)getObjectOfClass:(Class)cls forKey:(NSString *)key {
    return m_mmkv->getObject(key, cls);
}

- (BOOL)getBoolForKey:(NSString *)key {
    return [self getBoolForKey:key defaultValue:FALSE];
}
- (BOOL)getBoolForKey:(NSString *)key defaultValue:(BOOL)defaultValue {
    return m_mmkv->getBool(key, defaultValue);
}

- (int32_t)getInt32ForKey:(NSString *)key {
    return [self getInt32ForKey:key defaultValue:0];
}
- (int32_t)getInt32ForKey:(NSString *)key defaultValue:(int32_t)defaultValue {
    return m_mmkv->getInt32(key, defaultValue);
}

- (uint32_t)getUInt32ForKey:(NSString *)key {
    return [self getUInt32ForKey:key defaultValue:0];
}
- (uint32_t)getUInt32ForKey:(NSString *)key defaultValue:(uint32_t)defaultValue {
    return m_mmkv->getUInt32(key, defaultValue);
}

- (int64_t)getInt64ForKey:(NSString *)key {
    return [self getInt64ForKey:key defaultValue:0];
}
- (int64_t)getInt64ForKey:(NSString *)key defaultValue:(int64_t)defaultValue {
    return m_mmkv->getInt64(key, defaultValue);
}

- (uint64_t)getUInt64ForKey:(NSString *)key {
    return [self getUInt64ForKey:key defaultValue:0];
}
- (uint64_t)getUInt64ForKey:(NSString *)key defaultValue:(uint64_t)defaultValue {
    return m_mmkv->getUInt64(key, defaultValue);
}

- (float)getFloatForKey:(NSString *)key {
    return [self getFloatForKey:key defaultValue:0];
}
- (float)getFloatForKey:(NSString *)key defaultValue:(float)defaultValue {
    return m_mmkv->getFloat(key, defaultValue);
}

- (double)getDoubleForKey:(NSString *)key {
    return [self getDoubleForKey:key defaultValue:0];
}
- (double)getDoubleForKey:(NSString *)key defaultValue:(double)defaultValue {
    return m_mmkv->getDouble(key, defaultValue);
}

- (nullable NSString *)getStringForKey:(NSString *)key {
    return [self getStringForKey:key defaultValue:nil];
}
- (nullable NSString *)getStringForKey:(NSString *)key defaultValue:(nullable NSString *)defaultValue {
    if (key.length <= 0) {
        return defaultValue;
    }
    NSString *valueString = [self getObjectOfClass:NSString.class forKey:key];
    if (!valueString) {
        valueString = defaultValue;
    }
    return valueString;
}

- (nullable NSDate *)getDateForKey:(NSString *)key {
    return [self getDateForKey:key defaultValue:nil];
}
- (nullable NSDate *)getDateForKey:(NSString *)key defaultValue:(nullable NSDate *)defaultValue {
    if (key.length <= 0) {
        return defaultValue;
    }
    NSDate *valueDate = [self getObjectOfClass:NSDate.class forKey:key];
    if (!valueDate) {
        valueDate = defaultValue;
    }
    return valueDate;
}

- (nullable NSData *)getDataForKey:(NSString *)key {
    return [self getDataForKey:key defaultValue:nil];
}
- (nullable NSData *)getDataForKey:(NSString *)key defaultValue:(nullable NSData *)defaultValue {
    if (key.length <= 0) {
        return defaultValue;
    }
    NSData *valueData = [self getObjectOfClass:NSData.class forKey:key];
    if (!valueData) {
        valueData = defaultValue;
    }
    return valueData;
}

- (size_t)getValueSizeForKey:(NSString *)key {
    return m_mmkv->getValueSize(key, false);
}

- (int32_t)writeValueForKey:(NSString *)key toBuffer:(NSMutableData *)buffer {
    return m_mmkv->writeValueToBuffer(key, buffer.mutableBytes, static_cast<int32_t>(buffer.length));
}

#pragma mark - enumerate

- (BOOL)containsKey:(NSString *)key {
    return m_mmkv->containsKey(key);
}

- (size_t)count {
    return m_mmkv->count();
}

- (size_t)totalSize {
    return m_mmkv->totalSize();
}

- (size_t)actualSize {
    return m_mmkv->actualSize();
}

- (void)enumerateKeys:(void (^)(NSString *key, BOOL *stop))block {
    m_mmkv->enumerateKeys(block);
}

- (NSArray *)allKeys {
    return m_mmkv->allKeys();
}

- (void)removeValueForKey:(NSString *)key {
    m_mmkv->removeValueForKey(key);
}

- (void)removeValuesForKeys:(NSArray *)arrKeys {
    m_mmkv->removeValuesForKeys(arrKeys);
}

- (void)sync {
    m_mmkv->sync(mmkv::MMKV_SYNC);
}

- (void)async {
    m_mmkv->sync(mmkv::MMKV_ASYNC);
}

- (void)checkContentChanged {
    m_mmkv->checkContentChanged();
}

+ (void)onAppTerminate {
    g_lock->lock();

    // make sure no further call will go into m_mmkv
    [g_instanceDic enumerateKeysAndObjectsUsingBlock:^(id _Nonnull key, MMKV *_Nonnull mmkv, BOOL *_Nonnull stop) {
        mmkv->m_mmkv = nullptr;
    }];
    [g_instanceDic release];
    g_instanceDic = nil;

    [g_basePath release];
    g_basePath = nil;

    [g_groupPath release];
    g_groupPath = nil;

    mmkv::MMKV::onExit();

    g_lock->unlock();
    delete g_lock;
    g_lock = nullptr;
}

static bool g_isAutoCleanUpEnabled = false;
static uint32_t g_maxIdleSeconds = 0;
static dispatch_source_t g_autoCleanUpTimer = nullptr;

+ (void)enableAutoCleanUp:(uint32_t)maxIdleMinutes NS_SWIFT_NAME(enableAutoCleanUp(maxIdleMinutes:)) {
    MMKVInfo("enable auto clean up with maxIdleMinutes:%zu", maxIdleMinutes);
    SCOPED_LOCK(g_lock);

    g_isAutoCleanUpEnabled = true;
    g_maxIdleSeconds = maxIdleMinutes * 60;

    if (!g_autoCleanUpTimer) {
        g_autoCleanUpTimer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0));
        dispatch_source_set_event_handler(g_autoCleanUpTimer, ^{
            [MMKV tryAutoCleanUpInstances];
        });
    } else {
        dispatch_suspend(g_autoCleanUpTimer);
    }
    dispatch_source_set_timer(g_autoCleanUpTimer, dispatch_time(DISPATCH_TIME_NOW, g_maxIdleSeconds * NSEC_PER_SEC), g_maxIdleSeconds * NSEC_PER_SEC, 0);
    dispatch_resume(g_autoCleanUpTimer);
}

+ (void)disableAutoCleanUp {
    MMKVInfo("disable auto clean up");
    SCOPED_LOCK(g_lock);

    g_isAutoCleanUpEnabled = false;
    g_maxIdleSeconds = 0;

    if (g_autoCleanUpTimer) {
        dispatch_source_cancel(g_autoCleanUpTimer);
        dispatch_release(g_autoCleanUpTimer);
        g_autoCleanUpTimer = nullptr;
    }
}

+ (void)tryAutoCleanUpInstances {
    SCOPED_LOCK(g_lock);

#ifdef MMKV_IOS
    if (mmkv::MMKV::isInBackground()) {
        MMKVInfo("don't cleanup in background, might just wakeup from suspend");
        return;
    }
#endif

    uint64_t now = [NSDate timeIntervalSinceReferenceDate];
    NSMutableArray *arrKeys = [NSMutableArray array];
    [g_instanceDic enumerateKeysAndObjectsUsingBlock:^(id _Nonnull key, id _Nonnull obj, BOOL *_Nonnull stop) {
        auto mmkv = (MMKV *) obj;
        if (mmkv->m_lastAccessTime + g_maxIdleSeconds <= now && mmkv.retainCount == 1) {
            [arrKeys addObject:key];
        }
    }];
    if (arrKeys.count > 0) {
        auto msg = [NSString stringWithFormat:@"auto cleanup mmkv %@", arrKeys];
        MMKVInfo(msg.UTF8String);
        [g_instanceDic removeObjectsForKeys:arrKeys];
    }
}

+ (NSString *)mmkvBasePath {
    if (g_basePath.length > 0) {
        return g_basePath;
    }

    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *documentPath = (NSString *) [paths firstObject];
    if ([documentPath length] > 0) {
        g_basePath = [[documentPath stringByAppendingPathComponent:@"mmkv"] retain];
        return g_basePath;
    } else {
        return @"";
    }
}

+ (void)setMMKVBasePath:(NSString *)basePath {
    if (basePath.length > 0) {
        [g_basePath release];
        g_basePath = [basePath retain];
        [MMKV initializeMMKV:basePath];

        // still warn about it
        g_hasCalledInitializeMMKV = NO;

        MMKVInfo("set MMKV base path to: %@", g_basePath);
    }
}

static NSString *md5(NSString *value) {
    uint8_t md[MD5_DIGEST_LENGTH] = {};
    char tmp[3] = {}, buf[33] = {};
    auto data = [value dataUsingEncoding:NSUTF8StringEncoding];
    openssl::MD5((uint8_t *) data.bytes, data.length, md);
    for (auto ch : md) {
        snprintf(tmp, sizeof(tmp), "%2.2x", ch);
        strcat(buf, tmp);
    }
    return [NSString stringWithCString:buf encoding:NSASCIIStringEncoding];
}

+ (NSString *)mmapKeyWithMMapID:(NSString *)mmapID rootPath:(nullable NSString *)rootPath {
    NSString *string = nil;
    if ([rootPath length] > 0 && [rootPath isEqualToString:[MMKV mmkvBasePath]] == NO) {
        string = md5([rootPath stringByAppendingPathComponent:mmapID]);
    } else {
        string = mmapID;
    }
    MMKVDebug("mmapKey: %@", string);
    return string;
}

+ (BOOL)isFileValid:(NSString *)mmapID {
    return [self isFileValid:mmapID rootPath:nil];
}

+ (BOOL)isFileValid:(NSString *)mmapID rootPath:(nullable NSString *)path {
    if (mmapID.length > 0) {
        if (path.length > 0) {
            string rootPath(path.UTF8String);
            return mmkv::MMKV::isFileValid(mmapID.UTF8String, &rootPath);
        } else {
            return mmkv::MMKV::isFileValid(mmapID.UTF8String, nullptr);
        }
    }
    return NO;
}

+ (void)registerHandler:(id<MMKVHandler>)handler {
    SCOPED_LOCK(g_lock);
    g_callbackHandler = handler;

    if ([g_callbackHandler respondsToSelector:@selector(mmkvLogWithLevel:file:line:func:message:)]) {
        g_isLogRedirecting = true;
        mmkv::MMKV::registerLogHandler(LogHandler);
    }
    if ([g_callbackHandler respondsToSelector:@selector(onMMKVCRCCheckFail:)] ||
        [g_callbackHandler respondsToSelector:@selector(onMMKVFileLengthError:)]) {
        mmkv::MMKV::registerErrorHandler(ErrorHandler);
    }
    if ([g_callbackHandler respondsToSelector:@selector(onMMKVContentChange:)]) {
        mmkv::MMKV::registerContentChangeHandler(ContentChangeHandler);
    }
}

+ (void)unregiserHandler {
    SCOPED_LOCK(g_lock);

    g_isLogRedirecting = false;
    g_callbackHandler = nil;

    mmkv::MMKV::unRegisterLogHandler();
    mmkv::MMKV::unRegisterErrorHandler();
    mmkv::MMKV::unRegisterContentChangeHandler();
}

+ (void)setLogLevel:(MMKVLogLevel)logLevel {
    mmkv::MMKV::setLogLevel((mmkv::MMKVLogLevel) logLevel);
}

- (uint32_t)migrateFromUserDefaults:(NSUserDefaults *)userDaults {
    @autoreleasepool {
        NSDictionary *dic = [userDaults dictionaryRepresentation];
        if (dic.count <= 0) {
            MMKVInfo("migrate data fail, userDaults is nil or empty");
            return 0;
        }
        __block uint32_t count = 0;
        [dic enumerateKeysAndObjectsUsingBlock:^(id _Nonnull key, id _Nonnull obj, BOOL *_Nonnull stop) {
            if ([key isKindOfClass:[NSString class]]) {
                NSString *stringKey = key;
                if ([MMKV tranlateData:obj key:stringKey kv:self]) {
                    count++;
                }
            } else {
                MMKVWarning("unknown type of key:%@", key);
            }
        }];
        return count;
    }
}

+ (BOOL)tranlateData:(id)obj key:(NSString *)key kv:(MMKV *)kv {
    if ([obj isKindOfClass:[NSString class]]) {
        return [kv setString:obj forKey:key];
    } else if ([obj isKindOfClass:[NSData class]]) {
        return [kv setData:obj forKey:key];
    } else if ([obj isKindOfClass:[NSDate class]]) {
        return [kv setDate:obj forKey:key];
    } else if ([obj isKindOfClass:[NSNumber class]]) {
        NSNumber *num = obj;
        CFNumberType numberType = CFNumberGetType((CFNumberRef) obj);
        switch (numberType) {
            case kCFNumberCharType:
            case kCFNumberSInt8Type:
            case kCFNumberSInt16Type:
            case kCFNumberSInt32Type:
            case kCFNumberIntType:
            case kCFNumberShortType:
                return [kv setInt32:num.intValue forKey:key];
            case kCFNumberSInt64Type:
            case kCFNumberLongType:
            case kCFNumberNSIntegerType:
            case kCFNumberLongLongType:
                return [kv setInt64:num.longLongValue forKey:key];
            case kCFNumberFloat32Type:
                return [kv setFloat:num.floatValue forKey:key];
            case kCFNumberFloat64Type:
            case kCFNumberDoubleType:
                return [kv setDouble:num.doubleValue forKey:key];
            default:
                MMKVWarning("unknown number type:%ld, key:%@", (long) numberType, key);
                return NO;
        }
    } else if ([obj isKindOfClass:[NSArray class]] || [obj isKindOfClass:[NSDictionary class]]) {
        return [kv setObject:obj forKey:key];
    } else {
        MMKVWarning("unknown type of key:%@", key);
    }
    return NO;
}

@end

#pragma makr - callbacks

static void LogHandler(mmkv::MMKVLogLevel level, const char *file, int line, const char *function, NSString *message) {
    [g_callbackHandler mmkvLogWithLevel:(MMKVLogLevel) level file:file line:line func:function message:message];
}

static mmkv::MMKVRecoverStrategic ErrorHandler(const string &mmapID, mmkv::MMKVErrorType errorType) {
    if (errorType == mmkv::MMKVCRCCheckFail) {
        if ([g_callbackHandler respondsToSelector:@selector(onMMKVCRCCheckFail:)]) {
            auto ret = [g_callbackHandler onMMKVCRCCheckFail:[NSString stringWithUTF8String:mmapID.c_str()]];
            return (mmkv::MMKVRecoverStrategic) ret;
        }
    } else {
        if ([g_callbackHandler respondsToSelector:@selector(onMMKVFileLengthError:)]) {
            auto ret = [g_callbackHandler onMMKVFileLengthError:[NSString stringWithUTF8String:mmapID.c_str()]];
            return (mmkv::MMKVRecoverStrategic) ret;
        }
    }
    return mmkv::OnErrorDiscard;
}

static void ContentChangeHandler(const string &mmapID) {
    if ([g_callbackHandler respondsToSelector:@selector(onMMKVContentChange:)]) {
        [g_callbackHandler onMMKVContentChange:[NSString stringWithUTF8String:mmapID.c_str()]];
    }
}


MMKV* getInstance(const char* mapId)
{
    NSString* pMapId = [NSString stringWithUTF8String:mapId];
    if (pMapId != nil && pMapId.length > 0)
    {
        return [MMKV mmkvWithID:pMapId];
    }
    return [MMKV defaultMMKV];
}

#if defined(__cplusplus)
extern "C" {
#endif

void __init(const char* path, int level)
{
    NSString* root = [NSString stringWithUTF8String:path];
    
    [MMKV initializeMMKV:root logLevel:(MMKVLogLevel)level];
}

void __encodeString(const char* mapId, const char* key, const char* value)
{
    MMKV* mmkv = [MMKV getInstance:mapId];
    NSString* pKey = [NSString stringWithUTF8String:key];
    NSString* pValue = [NSString stringWithUTF8String:value];
    if (mmkv != nil)
    {
        [mmkv setString:pValue forKey:pKey];
    }
}

const char* __decodeString(const char* mapId, const char* key)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        NSString* pKey = [NSString stringWithUTF8String:key];
        auto value = [mmkv getStringForKey:pKey];
        if (value == nil)
        {
            return nullptr;
        }
        return [value cStringUsingEncoding:NSUTF8StringEncoding];
    }
    return nullptr;
}

void __encodeBool(const char* mapId, const char* key, const bool value)
{
    MMKV* mmkv = [MMKV getInstance:mapId];
    NSString* pKey = [NSString stringWithUTF8String:key];
    if (mmkv != nil)
    {
        [mmkv setBool:value forKey:pKey];
    }
}

bool __decodeBool(const char* mapId, const char* key, const bool defaultValue)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        NSString* pKey = [NSString stringWithUTF8String:key];
        return [mmkv getBoolForKey:pKey defaultValue:defaultValue];
    }
    return defaultValue;
}

void __encodeInt(const char* mapId, const char* key, const int value)
{
    MMKV* mmkv = [MMKV getInstance:mapId];
    NSString* pKey = [NSString stringWithUTF8String:key];
    if (mmkv != nil)
    {
        [mmkv setInt32:value forKey:pKey];
    }
}

int32_t __decodeInt(const char* mapId, const char* key, const int32_t defaultValue)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        NSString* pKey = [NSString stringWithUTF8String:key];
        return [mmkv getInt32ForKey:pKey defaultValue:defaultValue];
    }
    return defaultValue;
}

void __encodeLong(const char* mapId, const char* key, const long value)
{
    MMKV* mmkv = [MMKV getInstance:mapId];
    NSString* pKey = [NSString stringWithUTF8String:key];
    if (mmkv != nil)
    {
        [mmkv setInt64:value forKey:pKey];
    }
}

int64_t __decodeLong(const char* mapId, const char* key, const int64_t defaultValue)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        NSString* pKey = [NSString stringWithUTF8String:key];
        return [mmkv getInt64ForKey:pKey defaultValue:defaultValue];
    }
    return defaultValue;
}

void __encodeFloat(const char* mapId, const char* key, const float value)
{
    MMKV* mmkv = [MMKV getInstance:mapId];
    NSString* pKey = [NSString stringWithUTF8String:key];
    if (mmkv != nil)
    {
        [mmkv setFloat:value forKey:pKey];
    }
}

float __decodeFloat(const char* mapId, const char* key, const float defaultValue)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        NSString* pKey = [NSString stringWithUTF8String:key];
        return [mmkv getFloatForKey:pKey defaultValue:defaultValue];
    }
    return defaultValue;
}

void __encodeDouble(const char* mapId, const char* key, const double value)
{
    MMKV* mmkv = [MMKV getInstance:mapId];
    NSString* pKey = [NSString stringWithUTF8String:key];
    if (mmkv != nil)
    {
        [mmkv setDouble:value forKey:pKey];
    }
}

double __decodeDouble(const char* mapId, const char* key, const double defaultValue)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        NSString* pKey = [NSString stringWithUTF8String:key];
        return [mmkv getDoubleForKey:pKey defaultValue:defaultValue];
    }
    return defaultValue;
}

size_t __totalSize(const char* mapId)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        return [mmkv totalSize];
    }
    return 0;
}

size_t __actualSize(const char* mapId)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        return [mmkv actualSize];
    }
    return 0;
}

void __removeValueForKey(const char* mapId, const char* key)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        NSString* pKey = [NSString stringWithUTF8String:key];
        [mmkv removeValueForKey:pKey];
    }
}

void __removeValuesForKeys(const char* mapId, const char** keys, const int size)
{
    if (keys == nullptr)
    {
        return;
    }
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        NSMutableArray* array = [NSMutableArray array];
        for (int i = 0; i < size; i++) {
            NSString* str = [NSString stringWithUTF8String:keys[i]];
            [array addObject:str];
        }
        
        [mmkv removeValuesForKeys:array];
    }
}

bool __containsKey(const char* mapId, const char* key)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        NSString* pKey = [NSString stringWithUTF8String:key];
        return [mmkv containsKey:pKey];
    }
    
    return false;
}

const char* __getAllKey(const char* mapId)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        NSArray* array = [mmkv allKeys];
              
        NSString* result = [array componentsJoinedByString:@","];
        
        return [result cStringUsingEncoding:NSUTF8StringEncoding];
    }
    
    return nullptr;
}

void __sync(const char* mapId)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        [mmkv sync];
    }
}

void __async(const char* mapId)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        [mmkv async];
    }
}

void __clearAll(const char* mapId)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        [mmkv clearAll];
    }
}

void __close(const char* mapId)
{
    auto mmkv = [MMKV getInstance:mapId];
    if (mmkv != nil)
    {
        [mmkv close];
    }
}


#if defined(__cplusplus)
}
#endif

```

至此，Android、iOS、Win32的导出接口介绍完毕

1. Android输出的是三个架构的静态库libmmkv.so（放到Plugins/Android目录）
2. iOS输出arm64的静态库libmmkv.a（放到Plugins/iOS目录）
3. Win32输出动态链接库mmkv.dll（放到Plugins/x86_64目录)

在unity方使用的接口定义如下

```c#

using System;
using System.Collections.Generic;
using System.Runtime.InteropServices;
using LuaInterface;
using MoonCommonLib;

namespace MoonClient.SDKManager
{
    
    public static class MMMKV
    {
        [NoToLua]
        public enum EMMKVLogLevel
        {
            Debug = 0, // not available for release/product build
            Info = 1,  // default level
            Warning,
            Error,
            None, // special level used to disable all log messages
        };
#if UNITY_IOS
        internal const string LibName = "__Internal";
#else
        internal const string LibName = "mmkv";
#endif
        internal const string DefaultMapId = "common";

        [NoToLua]
        [DllImport(LibName)]
        private static extern void __init(string path, int logLevel);
        [NoToLua]
        [DllImport(LibName)]
        private static extern void __encodeString(string mapId, string key, string value);
        [NoToLua]
        [DllImport(LibName)]
        private static extern IntPtr __decodeString(string mapId, string key);
        [NoToLua]
        [DllImport(LibName)]
        private static extern void __encodeInt(string mapId, string key, int value);
        [NoToLua]
        [DllImport(LibName)]
        private static extern int __decodeInt(string mapId, string key, int defaultValue);
        [NoToLua]
        [DllImport(LibName)]
        private static extern void __encodeLong(string mapId, string key, long value);
        [NoToLua]
        [DllImport(LibName)]
        private static extern long __decodeLong(string mapId, string key, long defaultValue);
        [NoToLua]
        [DllImport(LibName)]
        private static extern void __encodeFloat(string mapId, string key, float value);
        [NoToLua]
        [DllImport(LibName)]
        private static extern float __decodeFloat(string mapId, string key, float defaultValue);
        [NoToLua]
        [DllImport(LibName)]
        private static extern void __encodeDouble(string mapId, string key, double value);
        [NoToLua]
        [DllImport(LibName)]
        private static extern double __decodeDouble(string mapId, string key, double defaultValue);
        [NoToLua]
        [DllImport(LibName)]
        private static extern void __encodeBool(string mapId, string key, bool value);
        [NoToLua]
        [DllImport(LibName)]
        private static extern bool __decodeBool(string mapId, string key, bool defaultValue);
        [NoToLua]
        [DllImport(LibName)]
        private static extern ulong __totalSize(string mapId);
        [NoToLua]
        [DllImport(LibName)]
        private static extern ulong __actualSize(string mapId);
        [NoToLua]
        [DllImport(LibName)]
        private static extern void __removeValueForKey(string mapId, string key);
        [NoToLua]
        [DllImport(LibName)]
        private static extern void __removeValuesForKeys(string mapId, string[] keys, int size);
        [NoToLua]
        [DllImport(LibName)]
        private static extern bool __containsKey(string mapId, string key);
        [NoToLua]
        [DllImport(LibName)]
        private static extern IntPtr __getAllKey(string mapId);
        [NoToLua]
        [DllImport(LibName)]
        private static extern void __sync(string mapId);
        [NoToLua]
        [DllImport(LibName)]
        private static extern void __async(string mapId);
        [NoToLua]
        [DllImport(LibName)]
        private static extern void __clearAll(string mapId);
        [NoToLua]
        [DllImport(LibName)]
        private static extern void __close(string mapId);
#if !UNITY_IOS
        [NoToLua]
        [DllImport(LibName)]
        private static extern void __freeNewMemory(IntPtr buffer);
#endif
        static MMMKV()
        {
            Init(EMMKVLogLevel.Info);
        }

        [NoToLua]
        public static void Test()
        {
            SetString("test1", "hello mmkv");
            MDebug.singleton.AddLog($"MMKV test GetString:{GetString("test1", "defaultValue")}");
        }

        public static void Init(EMMKVLogLevel level)
        {
            __init(DirectoryEx.Combine(PathEx.PersistentDataPath, "mmkv"), (int)level);
        }

        public static void SetString(string key, string value, string mapId = DefaultMapId)
        {
            __encodeString(mapId, key, value);
        }

        public static string GetString(string key, string defaultValue = "", string mapId = DefaultMapId)
        {
            IntPtr ptr = __decodeString(mapId, key);
            if (ptr == IntPtr.Zero)
            {
                return defaultValue;
            }
            var unicode = Marshal.PtrToStringAnsi(ptr);
#if !UNITY_IOS
            __freeNewMemory(ptr);
#endif
            return unicode;
        }

        public static void SetBool(string key, bool value, string mapId = DefaultMapId)
        {
            __encodeBool(mapId, key, value);
        }

        public static bool GetBool(string key, bool defaultValue = false, string mapId = DefaultMapId)
        {
            return __decodeBool(mapId, key, defaultValue);
        }

        public static void SetInt(string key, int value, string mapId = DefaultMapId)
        {
            __encodeInt(mapId, key, value);
        }

        public static int GetInt(string key, int defaultValue = 0, string mapId = DefaultMapId)
        {
            return __decodeInt(mapId, key, defaultValue);
        }

        public static void SetLong(string key, long value, string mapId = DefaultMapId)
        {
            __encodeLong(mapId, key, value);
        }

        public static long GetLong(string key, long defaultValue = 0L, string mapId = DefaultMapId)
        {
            return __decodeLong(mapId, key, defaultValue);
        }

        public static void SetFloat(string key, float value, string mapId = DefaultMapId)
        {
            __encodeFloat(mapId, key, value);
        }

        public static float GetFloat(string key, float defaultValue = 0f, string mapId = DefaultMapId)
        {
            return __decodeFloat(mapId, key, defaultValue);
        }

        public static void SetDouble(string key, double value, string mapId = DefaultMapId)
        {
            __encodeDouble(mapId, key, value);
        }

        public static double GetDouble(string key, double defaultValue = 0.0f, string mapId = DefaultMapId)
        {
            return __decodeDouble(mapId, key, defaultValue);
        }

        public static void RemoveKey(string key, string mapId = DefaultMapId)
        {
            __removeValueForKey(mapId, key);
        }

        public static void RemoveKeys(string[] keys, string mapId = DefaultMapId)
        {
            __removeValuesForKeys(mapId, keys, keys.Length);
        }

        public static bool Contains(string key, string mapId = DefaultMapId)
        {
            return __containsKey(mapId, key);
        }

        public static ulong GetTotalSize(string mapId = DefaultMapId)
        {
            return __totalSize(mapId);
        }

        public static ulong GetActualSize(string mapId = DefaultMapId)
        {
            return __actualSize(mapId);
        }

        public static List<string> GetAllKeys(string mapId = DefaultMapId)
        {
            var list = new List<string>();
            var ptr = __getAllKey(mapId);
            string ret;
            if (ptr == IntPtr.Zero)
            {
                ret = string.Empty;
            }
            else
            {
                ret = Marshal.PtrToStringAnsi(ptr);
#if !UNITY_IOS
                __freeNewMemory(ptr);
#endif
            }
            if (!string.IsNullOrEmpty(ret))
            {
                var set = ret.Split(new[] { ',' });
                list.AddRange(set);
            }
            return list;
        }

        public static void Sync(string mapId = DefaultMapId)
        {
            __sync(mapId);
        }

        public static void Async(string mapId = DefaultMapId)
        {
            __async(mapId);
        }

        public static void ClearAll(string mapId = DefaultMapId)
        {
            __clearAll(mapId);
        }

        public static void Close(string mapId = DefaultMapId)
        {
            __close(mapId);
        }
    }
}

```

直接使用即可。

当然，如果有对cmake熟悉的大佬，上方也不用那么麻烦，直接对着Core工程操作即可。







