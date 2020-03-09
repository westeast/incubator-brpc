## brpc的依赖问题
error
```
undifined refrences to google protobuf之类的错误 多个
CMakeFiles/protoc-gen-mcpack.dir/mcpack2pb/generator.cpp.o: In function `google::protobuf::internal::StringTypeTraits::Get(int, google::protobuf::internal::ExtensionSet const&, std::string const&)':
/usr/include/google/protobuf/extension_set.h:789: undefined reference to `google::protobuf::internal::ExtensionSet::GetString(int, std::string const&) const'
CMakeFiles/protoc-gen-mcpack.dir/mcpack2pb/generator.cpp.o: In function `mcpack2pb::generate_declarations(std::set<std::string, std::less<std::string>, std::allocator<std::string> > const&, std::set<std::string, std::less<std::string>, std::allocator<std::string> > const&, google::protobuf::io::Printer&)':
/paddle/incubator-brpc/src/mcpack2pb/generator.cpp:196: undefined reference to `google::protobuf::io::Printer::Print(char const*, char const*, std::string const&)'
/paddle/incubator-brpc/src/mcpack2pb/generator.cpp:203: undefined reference to `google::protobuf::io::Printer::Print(char const*, char const*, std::string const&)'
CMakeFiles/protoc-gen-mcpack.dir/mcpack2pb/generator.cpp.o: In function `mcpack2pb::generate_parsing(google::protobuf::Descriptor const*, std::set<std::string, std::less<std::string>, std::allocator<std::string> >&, std::set<std::string, std::less<std::string>, std::allocator<std::string> >&, google::protobuf::io::Printer&)':
/paddle/incubator-brpc/src/mcpack2pb/generator.cpp:276: undefined reference to `google::protobuf::io::Printer::Print(char const*, char const*, std::string const&)'
/paddle/incubator-brpc/src/mcpack2pb/generator.cpp:492: undefined reference to `google::protobuf::io::Printer::Print(char const*, char const*, std::string const&, char const*, std::string const&, char const*, std::string const&)'
/paddle/incubator-brpc/src/mcpack2pb/generator.cpp:496: undefined reference to `google::protobuf::io::Printer::Print(char const*, char const*, std::string const&, char const*, std::string const&)'
/paddle/incubator-brpc/src/mcpack2pb/generator.cpp:578: undefined reference to `google::protobuf::io::Printer::Print(char const*, char const*, std::string const&, char const*, std::string const&, char const*, std::string const&)'
/paddle/incubator-brpc/src/mcpack2pb/generator.cpp:584: undefined reference to `google::protobuf::io::Printer::Print(char const*, char const*, std::string const&)'
```
要使用proto3,重新 下载 并编译安装proto3 make make install 后，执行protoc还是2.6，发现丙个protoc都在/usr下，执行apt-get remove libprotoc-dev 再执行protoc成功，现在系统里还有2.6的lib还没有删除.  libprotobuf  libgflags-dev 安装的这两个也删除了
https://www.cnblogs.com/timeddd/p/11081031.html

重新编译了gflags后
```

usr/local/bin/ld: /usr/local/lib/libgflags.a(gflags.cc.o): relocation R_X86_64_32S against `.rodata' can not be used when making a shared object; recompile with -fPIC
/usr/local/bin/ld: /usr/local/lib/libgflags.a(gflags_reporting.cc.o): relocation R_X86_64_32 against `.rodata.str1.1' can not be used when making a shared object; recompile with -fPIC
/usr/local/bin/ld: /usr/local/lib/libgflags.a(gflags_completions.cc.o): relocation R_X86_64_32S against symbol `_ZNSs4_Rep20_S_empty_rep_storageE@@GLIBCXX_3.4' can not be used when making a shared object; recompile with -fPIC
```
进入anyQ中下载的gflags中来安装，注意不是camke ..不然生成的是静态库
cmake -DCMAKE_INSTALL_PREFIX=/usr/local -DBUILD_SHARED_LIBS=ON -DGFLAGS_NAMESPACE=google -G"Unix Makefiles" ../
make && make install 
brpc编译成功，参考这里gflags的编译：https://blog.csdn.net/lanyuxuan100/article/details/69633785


## 运行sample
上述编译完成后只生成了，想着的libbrpc.a及libbrpc.so及相关的一些工具
运行一个http的例子
进入：example/http_c++/用cmake编译，有报错
/paddle/incubator-brpc/src/brpc/span.cpp:488: undefined reference to `leveldb::DB::Open(leveldb::Options const&, std::string const&, leveldb::DB**)'
/paddle/incubator-brpc/src/brpc/span.cpp:490: undefined reference to `leveldb::Status::ToString() const'
/paddle/incubator-brpc/src/brpc/span.cpp:497: undefined reference to `leveldb::DB::Open(leveldb::Options const&, std::string const&, leveldb::DB**)'

记录下levedb使用apt-get安装前后的变化
```
805db5ce8d1d /paddle/incubator-brpc/example/http_c++/build {master} find /usr -name *libleve* 
/usr/share/doc/libleveldb1v5
/usr/share/doc/libleveldb-dev
/usr/lib/x86_64-linux-gnu/libleveldb.so.1.18
/usr/lib/x86_64-linux-gnu/libleveldb.a
/usr/lib/x86_64-linux-gnu/libleveldb.so.1
/usr/lib/x86_64-linux-gnu/libleveldb.so
λ 805db5ce8d1d /paddle/incubator-brpc/example/http_c++/build {master} gpt^C
λ 805db5ce8d1d /paddle/incubator-brpc/example/http_c++/build {master} apt^C
λ 805db5ce8d1d /paddle/incubator-brpc/example/http_c++/build {master} apt-get remove libleveldb-dev
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  libgflags2v5 libleveldb1v5 libprotobuf-lite9v5
Use 'apt autoremove' to remove them.
The following packages will be REMOVED:
  libleveldb-dev
0 upgraded, 0 newly installed, 1 to remove and 79 not upgraded.
After this operation, 948 kB disk space will be freed.
Do you want to continue? [Y/n] Y
(Reading database ... 37131 files and directories currently installed.)
Removing libleveldb-dev:amd64 (1.18-5) ...
λ 805db5ce8d1d /paddle/incubator-brpc/example/http_c++/build {master} find /usr -name *libleve* 
/usr/share/doc/libleveldb1v5
/usr/lib/x86_64-linux-gnu/libleveldb.so.1.18
/usr/lib/x86_64-linux-gnu/libleveldb.so.1
λ 805db5ce8d1d /paddle/incubator-brpc/example/http_c++/build {master} 
```

计划使用AnyQ中下载的leveldb重新 安装。
进入目录 直接make 然后想着库copy过去，
make 
  427  cp -r include/* /usr/include/
  428  cp -r out-shared/lib* /usr/lib/
  429  cp -r out-static/lib* /usr/lib/
重新cmake 编译成功了。
现在的levedb
```
λ 805db5ce8d1d /paddle/incubator-brpc/example/http_c++/build {master} find /usr -name *libleve* 
/usr/share/doc/libleveldb1v5
/usr/lib/x86_64-linux-gnu/libleveldb.so.1.18
/usr/lib/x86_64-linux-gnu/libleveldb.so.1
/usr/lib/libleveldb.a
/usr/lib/libleveldb.so.1
/usr/lib/libleveldb.so.1.20
/usr/lib/libleveldb.so
```