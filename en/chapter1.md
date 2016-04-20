## Develop Enviroment Setup

[Protocol Buffers v3.0.0-beta-2](https://github.com/google/protobuf/releases/tag/v3.0.0-beta-2)

Released on Dec 31, 2015

```shell
libprotobuf.a, building for OSX, but linking in object file built for iOS, for architecture x86_64
```

[Protocol Buffers v2.5.0](https://github.com/google/protobuf/releases/tag/v2.5.0)

Released on Feb 28, 2013

 <br />

### Complie `libprotobuf.a`

[build-protobuf-ios](https://github.com/sinofool/build-protobuf-ios)

```shell
roor$ cd /usr/local/lib
roor$ ls -al
libprotobuf.a

root$ protoc --version
libprotoc 2.5.0
```

#### ARM architecture

-   x86_64

    ​Mac OS X

-   i386
      iPhone Simulator

-   armv6

    iPhone

    iPhone2

    iPhone3G

    iPod Touch

-   armv7

    iPhone3GS

    iPhone4

    iPhone4S

    iPad

    iPad2

    iPad3

    iPad mini

    iPod Touch 3G

    iPod Touch4

-   armv7s

    iPhone5

    iPhone5C

    iPad4

-   arm64

    iPhone5S

    iPhone6

    iPhone6 Plus

    iPhone6S

    iPad Air

    iPad mini2

`lipo thin`

``` shell
lipo xxx.a -thin [armv7|armv7s|arm64|i386|x86_64] -output xxx_thin.a
```

<br />

### Xcode Setup

#### Add Source

-   Add folder `protobuf-2.5.0/src/google` to `Xcode`

    ```C++
          - Classes
            - protobuf
                - google
                    - srouce code
                - lib
                    - libprotobuf.a
    ```

-   Remove files

    -   All `.proto`
    -   All `.cc`

-   Remove folder

    -   `testing` 
    -   `testdata` 

-   Modify file `stubs/platform_macros.h`

    ```C++
    // ...
    #elif defined(__ppc__)
    #define GOOGLE_PROTOBUF_ARCH_PPC 1
    #define GOOGLE_PROTOBUF_ARCH_32_BIT 1
    // add start
    #elif defined(__aarch64__)
    #define GOOGLE_PROTOBUF_ARCH_ARM 1
    #define GOOGLE_PROTOBUF_ARCH_64_BIT 1
    // add end
    // ...
    ```

<br />

#### Add `libprotobuf.a`

Add the `libprotobuf.a` to `Build Parses ==> Link Binary With Libraries`

<br />

#### Modify Search Path

-   `Build Settings == > Search Paths ==> User Header Search Paths`

                eg : `$(SRCROOT)/../Classes/protobuf`

-   `Build Settings == > Search Paths ==> Library Search Paths`

    eg : `$(SRCROOT)/../Classes/protobuf/lib`

<br />

## Message Proto

### Define Protos

`GameInfo.proto`

```C++
package game.info;

option java_package = "com.game.info";
option java_outer_classname = "GameInfoProtos";

import "GameEnum.proto";

// role info
message RoleInfo {
    required string name                    = 1;
    required enumeration.RoleType type      = 2;    
}

// item info
message ItemInfo {
    required string name                    = 1;
    optional int32 price                    = 2;
}

// game info
message GameInfo {
    repeated RoleInfo roleInfo              = 1;
    repeated ItemInfo itemInfo              = 2;
}
```

`GameEnum.proto`

```C++
package game.enumeration;

option java_package = "com.game.enumeration";
option java_outer_classname = "GameEnumProtos";

// enumeration type
enum RoleType {
    FIGHTER     = 1;
    BOWMAN      = 2;
    MAGICIAN    = 3;
}
```

<br />

### Compiler `.proto` 

```shell
protoc 
    --proto_path=IMPORT_PATH 
    --cpp_out=DST_DIR 
    --java_out=DST_DIR 
    --python_out=DST_DIR 
    path/to/file.proto
```

and will generate :

- `c++`
  - `GameEnum.pb.h`
  - `GameEnum.pb.cc`
  - `GameInfo.pb.h`
  - `GameInfo.pb.cc`

- java
- game
- enumeration
- `GameEnumProtos.java`
- info
- `GameInfoProtos.java`


-   `python`
    -   `GameEnum_pb2.py`
    -   `GameInfo_pb2.py`


-   `javascript`

            Only `.proto` needed.

<br />

## Serialize && Parse / `C++`

#### `Folder structure`

```C++
- CPP
    // proto generated files
    - GameEnum.pb.h
    - GameEnum.pb.cc
    - GameInfo.pb.h
    - GameInfo.pb.cc
    // .proto
    - proto
        - GameInfo.proto
        - GameEnum.proto
    // protobuf source code
    - google
        - protobuf
            - // ...
    // main test file
    - CPP_TEST.cpp
```

<br />

#### `Create Instance`

```C++
// Create a GameInfo object first
game::info::GameInfo gameInfo;

// add RoleInfo object to repeated field
game::info::RoleInfo *pRoleInfo1 = gameInfo.add_roleinfo();
pRoleInfo1->set_name("name1");
pRoleInfo1->set_type(game::enumeration::FIGHTER);

// add another RoleInfo object to repeated field
game::info::RoleInfo *pRoleInfo2 = gameInfo.add_roleinfo();
pRoleInfo2->set_name("name2");
pRoleInfo2->set_type(game::enumeration::BOWMAN);

// add ItemInfo object to repeated field
game::info::ItemInfo *pItemInfo1 = gameInfo.add_iteminfo();
pItemInfo1->set_name("item1");
pItemInfo1->set_price(100);

// add another ItemInfo object to repeated field
game::info::ItemInfo *pItemInfo2 = gameInfo.add_iteminfo();
pItemInfo2->set_name("item2");
pItemInfo2->set_price(200);
```

<br />

#### `Serialize to String`

``` C++
std::string gameInfoString;
gameInfo.SerializeToString(&gameInfoString);
```

<br />

#### `Serialize to File`

```C++
ofstream outputFile("GameInfo.bin", std::ostream::binary);
gameInfo.SerializeToOstream(&outputFile);
```

<br />

#### `Parse from String`

```C++
game::info::GameInfo gameInfo;
gameInfo.ParseFromString(gameInfoString);
```

<br />

#### `Parse from File`

```C++
game::info::GameInfo gameInfo;
ifstream is("GameInfo.bin", std::ifstream::binary);
gameInfo.ParseFromIstream(&is);
is.close();
```

<br />

#### `Dump GameInfo Instance`

```C++
// RoleInfo Array
for (int i = 0; i < gameInfo.roleinfo_size(); i++) {
    // role info
    game::info::RoleInfo roleInfo = gameInfo.roleinfo(i);
    printf("roleInfo: \n%s\n", roleInfo.Utf8DebugString().c_str());
}

// ItemInfo Array
for (int i = 0; i < gameInfo.iteminfo_size(); i++) {
    // role info
    game::info::ItemInfo itemInfo = gameInfo.iteminfo(i);
    printf("itemInfo: \n%s\n", itemInfo.Utf8DebugString().c_str());
}
```

<br />

#### `Relection Dump Message`

```C++
void dumpMessageReflection(const Message* pMessageInfo)
{
    // Descriptor
    const Descriptor* pDescriptor = pMessageInfo->GetDescriptor();
    // Reflection
    const Reflection* pReflection = pMessageInfo->GetReflection();
    // Fields
    for (int i = 0; i < pDescriptor->field_count(); i ++) {
        const FieldDescriptor* pMessageField = pDescriptor->field(i);
        int fieldSize = pMessageField->is_repeated() ? 
            pReflection->FieldSize(*pMessageInfo, pMessageField) : 0;
        // Field
        switch(pMessageField->type()) {
            case FieldDescriptor::TYPE_INT64 :
            case FieldDescriptor::TYPE_UINT64 :
            case FieldDescriptor::TYPE_FIXED64 :
            case FieldDescriptor::TYPE_FIXED32 :
            case FieldDescriptor::TYPE_BOOL :
            case FieldDescriptor::TYPE_GROUP :
            case FieldDescriptor::TYPE_BYTES :
            case FieldDescriptor::TYPE_UINT32 :
            case FieldDescriptor::TYPE_SFIXED32 :
            case FieldDescriptor::TYPE_SFIXED64 :
            case FieldDescriptor::TYPE_SINT32 :
            case FieldDescriptor::TYPE_SINT64 :
                printf("FieldDescriptor type not handled !");
                break;
            case FieldDescriptor::TYPE_INT32 :
                if (fieldSize == 0) {
                    printf("Value: %d\n\n", 
                        pReflection->GetInt32(*pMessageInfo, pMessageField)
                    );
                }
                else {
                    for (int j = 0; j < fieldSize; j ++) {
                        printf("Value: %d\n\n", 
                            pReflection->GetRepeatedInt32(
                                *pMessageInfo, pMessageField, j
                            )
                        );
                    }
                }
                break;
            case FieldDescriptor::TYPE_FLOAT :
                if (fieldSize == 0) {
                    printf("Value: %f\n\n", 
                        pReflection->GetFloat(*pMessageInfo, pMessageField)
                    );
                }
                else {
                    for (int j = 0; j < fieldSize; j ++) {
                        printf("Value: %f\n\n", 
                            pReflection->GetRepeatedFloat(
                                *pMessageInfo, pMessageField, j
                            )
                        );
                    }
                }
                break;
            case FieldDescriptor::TYPE_DOUBLE :
                if (fieldSize == 0) {
                    printf("Value: %lf\n\n", 
                        pReflection->GetDouble(*pMessageInfo, pMessageField)
                    );
                }
                else {
                    for (int j = 0; j < fieldSize; j ++) {
                        printf("Value: %lf\n\n", 
                            pReflection->GetRepeatedDouble(
                                *pMessageInfo, pMessageField, j
                            )
                        );
                    }
                }
                break;
            case FieldDescriptor::TYPE_STRING :
                if (fieldSize == 0) {
                    printf("Value: %s\n\n", 
                        pReflection->GetString(
                            *pMessageInfo, pMessageField
                        ).c_str()
                    );
                }
                else {
                    for (int j = 0; j < fieldSize; j ++) {
                        printf("Value: %s\n\n", 
                            pReflection->GetRepeatedString(
                                *pMessageInfo, pMessageField, j
                            ).c_str()
                        );
                    }
                }
                break;
            case FieldDescriptor::TYPE_ENUM :
                if (fieldSize == 0) {
                    printf("Value: %d\n\n", 
                        pReflection->GetEnum(
                            *pMessageInfo, pMessageField
                        )->number()
                    );
                }
                else {
                    for (int j = 0; j < fieldSize; j ++) {
                        printf("Value: %d\n\n", 
                            pReflection->GetRepeatedEnum(
                                *pMessageInfo, pMessageField, j
                            )->number()
                        );
                    }
                }
                break;
            case FieldDescriptor::TYPE_MESSAGE :
                // recurse
                if (fieldSize == 0) {
                    dumpMessageReflection(
                        &pReflection->GetMessage(*pMessageInfo, pMessageField)
                    );
                }
                else {
                    for (int j = 0; j < fieldSize; j ++) {
                        dumpMessageReflection(
                            &pReflection->GetRepeatedMessage(
                                *pMessageInfo, pMessageField, j
                            )
                        );
                    }
                }
                break;
            default:
                break;
        }
    }
}
```

<br />

#### `Relection Generate Message`

```C++
compiler::DiskSourceTree sourceTree;
sourceTree.MapPath("", "./proto");
compiler::Importer importer(&sourceTree, NULL);
importer.Import("GameEnum.proto");
importer.Import("GameInfo.proto");

DynamicMessageFactory factory;
const Descriptor* pDescriptorDynamic = NULL;
const Reflection* pReflectionDynamic = NULL;
const FieldDescriptor* pFieldDynamic = NULL;
const EnumDescriptor* pEnumDescriptorDynamic = NULL;
Message* pGameInfoDynamic = NULL;
Message* pRoleInfoDynamic = NULL;
Message* pItemInfoDynamic = NULL;

// create a instance of GameInfo
pDescriptorDynamic = importer.pool()->FindMessageTypeByName("game.info.GameInfo");
pGameInfoDynamic = factory.GetPrototype(pDescriptorDynamic)->New();
pReflectionDynamic = pGameInfoDynamic->GetReflection();

// get repeated roleinfo instance
pFieldDynamic = pDescriptorDynamic->FindFieldByName("roleInfo");
pRoleInfoDynamic = pReflectionDynamic->AddMessage(pGameInfoDynamic, pFieldDynamic);

// set string filed name
pDescriptorDynamic = importer.pool()->FindMessageTypeByName("game.info.RoleInfo");
pEnumDescriptorDynamic = 
    importer.pool()->FindEnumTypeByName("game.enumeration.RoleType");
pFieldDynamic = pDescriptorDynamic->FindFieldByName("name");
pRoleInfoDynamic->GetReflection()->SetString(
    pRoleInfoDynamic, pFieldDynamic, "role_name_dynamic1"
);
// set enum filed type
pFieldDynamic = pDescriptorDynamic->FindFieldByName("type");
pRoleInfoDynamic->GetReflection()->SetEnum(
    pRoleInfoDynamic, 
    pFieldDynamic, 
    pEnumDescriptorDynamic->FindValueByName("MAGICIAN")
);

// get repeated iteminfo instance
pDescriptorDynamic = importer.pool()->FindMessageTypeByName("game.info.GameInfo");
pFieldDynamic = pDescriptorDynamic->FindFieldByName("itemInfo");
pItemInfoDynamic = pReflectionDynamic->AddMessage(pGameInfoDynamic, pFieldDynamic);

// set string filed name
pDescriptorDynamic = importer.pool()->FindMessageTypeByName("game.info.ItemInfo");
pFieldDynamic = pDescriptorDynamic->FindFieldByName("name");
pItemInfoDynamic->GetReflection()->SetString(
    pItemInfoDynamic, pFieldDynamic, "item_name_dynamic1"
);
// set int32 filed type
pFieldDynamic = pDescriptorDynamic->FindFieldByName("price");
pItemInfoDynamic->GetReflection()->SetInt32(
    pItemInfoDynamic, pFieldDynamic, 999999
);
```

<br />

#### `Compiler && Run`

```shell
Comliper :
g++ -c -I./ GameInfo.pb.cc
g++ -c -I./ GameEnum.pb.cc
g++ -c -I./ CPP_TEST.cpp

Link :
g++ -L./ -lprotobuf GameEnum.pb.o GameInfo.pb.o CPP_TEST.o -o main

Compiler && Run :
g++ -I./ -L./ -lprotobuf GameInfo.pb.cc GameEnum.pb.cc CPP_TEST.cpp -o main

Run :
./main
```

<br />

### Serialize && Parse / `javascript`

>   https://github.com/dcodeIO/protobuf.js

#### Load Proto

```javascript
// cocos2d-js style
require("bytebuffer.js");
require("protobuf.js");
var ProtoBuf = dcodeIO.ProtoBuf;

// node.js style
// var ProtoBuf = require("./dist/protobuf");

var GameInfoBuilder = ProtoBuf.newBuilder();

// load .proto
ProtoBuf.loadProtoFile("../PROTO/GameInfo.proto", null, xappBuilder);
ProtoBuf.loadProtoFile("../PROTO/GameEnum.proto", null, xappBuilder);
```

<br />

#### Create Instance

```javascript
var Game = GameInfoBuilder.build("game");

// game
var gameInfoOut = new Game.info.GameInfo();

// role
var roleInfo1 = new Game.info.RoleInfo();
roleInfo1.name = "name1";
roleInfo1.type = Game.enumeration.RoleType.FIGHTER;
gameInfoOut.roleInfo.push(roleInfo1);

var roleInfo2 = new Game.info.RoleInfo();
roleInfo2.name = "name2";
roleInfo2.type = Game.enumeration.RoleType.BOWMAN;
gameInfoOut.roleInfo.push(roleInfo2);

// item
var itemInfo1 = new Game.info.ItemInfo();
itemInfo1.name = "item1";
itemInfo1.price = 100;
gameInfoOut.itemInfo.push(itemInfo1);

var itemInfo2 = new Game.info.ItemInfo();
itemInfo2.name = "item2";
itemInfo2.price = 200;
gameInfoOut.itemInfo.push(itemInfo2);
```

<br />

#### Encode / Decode

```javascript
// encode
var gameInfoData = gameInfoOut.encodeAB();

// decode
var gameInfoIn = Game.info.GameInfo.decode(gameInfoData, "utf8");

// dump
for (var i = 0; i < gameInfoIn.roleInfo.length; i ++) {
    var roleInfo = gameInfoIn.roleInfo[i];
    console.log(/* ... */);
}

for (var i = 0; i < gameInfoIn.itemInfo.length; i ++) {
    var itemInfo = gameInfoIn.itemInfo[i];
    console.log(/* ... */);
}
```

<br />

#### Load from `.bin`

```javascript
var gameInfo = new Game.info.GameInfo();
var content = ProtoBuf.Util.fetch("../BIN/RoleInfo.bin");
var gameInfo = Game.info.GameInfo.decode(content, "utf8");

for (var i = 0; i < gameInfo.roleInfo.length; i ++) {
    var roleInfo = gameInfo.roleInfo[i];
    console.log(/* ... */);
}

content = ProtoBuf.Util.fetch("../BIN/ItemInfo.bin");
gameInfo = Game.info.GameInfo.decode(content, "utf8");
for (var i = 0; i < gameInfo.itemInfo.length; i ++) {
    var itemInfo = gameInfo.itemInfo[i];
    console.log(/* ... */);
}
```

<br />

### Excel `.xls` to ProtoBuf `.bin` / `Python`

#### Excel Format 

excel : `ItemInfo.xls` 

sheet name: `ItemInfo`

| id   | name        | price |
| ---- | ----------- | ----- |
| 1    | item_name_1 | 111   |
| 2    | item_name_2 | 222   |
| 3    | item_name_3 | 333   |

excel : `RoleInfo.xls` 

sheet name: `RoleInfo`

| id   | name        | type |
| ---- | ----------- | ---- |
| 1    | role_name_1 | 1    |
| 2    | role_name_2 | 2    |
| 3    | role_name_3 | 3    |

<br />

#### Compiler `.proto`

``` python
# dir
PB_DIR = './PROTO'
PB_PY_DIR = './PYTHON'
XLS_DIR = './XLS'
BIN_DIR = './BIN'

'''
    eg:
        compile_protobuf(PB_DIR, PB_PY_DIR, 'a.proto', 'python')
        compile_protobuf(PB_DIR, PB_CPP_DIR, 'a.proto', 'cpp')
        compile_protobuf(PB_DIR, PB_JAVA_DIR, 'a.proto', 'java')
'''
def compile_protobuf(src, dst, proto_file, language):
    if language == 'cpp':
        out_language = 'cpp_out'
    elif language == 'python':
        out_language = 'python_out'
    elif language == 'java':
        out_language = 'java_out'
    else:
        out_language = 'python_out'
    os.system('protoc -I=%s --%s=%s %s' % (
        src, out_language, dst, src + '/' + proto_file)
    )
```

The output :

```C++
- PYTHON
    GameInfo_pb2.py
    GameEnum_pb2.py
```

<br />

#### Dynamic import

```python
import inspect

def dynamic_import():
    import PYTHON.GameInfo_pb2
    import_py = __import__('PYTHON.GameInfo_pb2')
    pb_attr = getattr(import_py, 'GameInfo_pb2')
    del import_py

    for name, obj in inspect.getmembers(PYTHON.GameInfo_pb2):
        if inspect.isclass(obj):
            import_pb_class(pb_attr, name)
            
def import_pb_class(pb_attr, class_name):
    attr = getattr(pb_attr, class_name)
    setattr(sys.modules[__name__], class_name, attr)

    descriptor = make_descriptor_name(class_name)
    attr = getattr(pb_attr, descriptor)
    setattr(sys.modules[__name__], descriptor, attr)
    
def make_descriptor_name(class_name):
    return '_' + class_name.upper()
```

<br />

#### Parse `.xls`

```python
def parse_xls_files():
    for worksheet_name, table in get_xls_files().items():
        message_object = class_from_string(worksheet_name)
        if not message_object:
            continue
        fields = list_fields(make_descriptor_name(worksheet_name))
        parse_table(worksheet_name, table, fields)
    
def class_from_string(class_name):
    try:
        instance = getattr(sys.modules[__name__], class_name)()
        return instance
    except:
        return None
```
<br />

#### List fields

```python
def list_fields(class_name):
    fields = []
    for field in getattr(sys.modules[__name__], class_name).fields:
        fields.append([field.name, field.type, field.index])
    return fields
```

<br />

#### Parse table

```python
def parse_table(worksheet_name, table, fields):
    ## all data will be put into a GameInfo instance
    gameInfo = class_from_string('GameInfo')
    for ri in range(len(table)):
        # ...
        row = table[ri]
        message = None
        for field in fields:
            value_index = get_value_index(table[0], field[0])
            # ...
            value = convert_pb_type(row[value_index][1], field[1])
            # ...
            message = getattr(
                gameInfo, worksheet_name[0].lower() + worksheet_name[1:]
            ).add() 
            setattr(message, field[0], value)
    # write to .bin
    # ...
    serializedObject = gameInfo.SerializeToString()
    pb_bin_file = "./bin/%s.bin" % worksheet_name
    pb_bin = open(pb_bin_file, "wb")
    pb_bin.write(serializedObject)
    pb_bin.close()
    
def convert_pb_type(value, type):
    if type in [FieldDescriptor.TYPE_INT64, 
                FieldDescriptor.TYPE_UINT64, 
                FieldDescriptor.TYPE_INT32, 
                FieldDescriptor.TYPE_ENUM]:
        return int(value)
    elif type in [FieldDescriptor.TYPE_DOUBLE, 
                  FieldDescriptor.TYPE_FLOAT]:
        return float(value)
    elif type in [FieldDescriptor.TYPE_STRING]:
        if isinstance(value, basestring):
            return value
        elif isinstance(value, float):
            return str(int(value))
        else:
            return str(value)
    else:
        return None
```

Generated `.bin`

```C++
// ItemInfo.bin
GameInfo
    repeated RoleInfo roleInfo =>
    {
        RoleInfo1,
        RoleInfo2,
        RoleInfo3,
        ...
    }
    repeated ItemInfo itemInfo => None

// RoleInfo.bin
GameInfo
    repeated RoleInfo roleInfo => None
    repeated ItemInfo itemInfo => 
    {
        ItemInfo1,
        ItemInfo2,
        ItemInfo3,
        ...
    }
```

<br />

#### `.bin` security

`.bin` can be decoded using `protoc` command.

##### decode with `proto`

```python
import sys
import subprocess

def decode(message_name, proto_name, bin_name):
    process = subprocess.Popen(
        ['/usr/local/bin/protoc', '--decode', message_name, proto_name],
        stdin = subprocess.PIPE,
        stdout = subprocess.PIPE,
        stderr = subprocess.PIPE
    )

    output = error = None
    try:
        f = open(bin_name, "rb")
        data = f.read()
        f.close()
        output, error = process.communicate(data)
        if error:
            print 'error: ' + error 
    except OSError:
        pass
    finally:
        if process.poll() != 0:
            process.wait()
    return output

print '------ result ------\n'
print 'message_name: ' + sys.argv[1]
print 'proto_name: ' + sys.argv[2]
print 'bin_name: ' + sys.argv[3]
print
print decode(sys.argv[1], sys.argv[2], sys.argv[3])
print '------ ------ ------\n'
```

Test with `ItemInfo.bin`

```shell
root$ python decode.py game.info.GameInfo GameInfo.proto ItemInfo.bin

------ result ------
message_name: game.info.GameInfo
proto_name: GameInfo.proto
bin_name: ItemInfo.bin

itemInfo {
  name: "item1"
  price: 100
}
itemInfo {
  name: "item2"
  price: 101
}
......
------ ------ ------
```

##### decode with `proto`

```python
import sys
import subprocess

def decode_raw(data):
    process = subprocess.Popen(
        ['/usr/local/bin/protoc', '--decode_raw'],
        stdin = subprocess.PIPE,
        stdout = subprocess.PIPE,
        stderr = subprocess.PIPE
    )
    output = error = None
    try:
        output, error = process.communicate(data)
    except OSError:
        pass
    finally:
        if process.poll() != 0:
            process.wait()
    return output

f = open(sys.argv[1], "rb")
data = f.read()
print '------ data ------\n'
print data
print '------ ------ ------\n'

print '------ result ------\n'
print decode_raw(data)
print '------ ------ ------\n'
f.close()
```

Test with `ItemInfo.bin`

```shell
root$ python decode_raw.py ItemInfo.bin 

------ data ------

    
item1d  
item2e  
item3f  
item4g  
item5h  
item6i  
item7j  
item8k  
item9l

item10m
------ ------ ------

------ result ------

2 {
  1: "item1"
  2: 100
}
2 {
  1: "item2"
  2: 101
}
2 {
  1: "item3"
  2: 102
}
......
------ ------ ------
```

##### decode raw data ( network data transfer )

>   [protoc-decode-lenprefix on Github](https://github.com/erikdw/protoc-decode-lenprefix)

<br />

### JSBinding

#### Send from `C++` 

```C++
JSObject* pGlobalObject = ScriptingCore::getInstance()->getGlobalObject();
JSContext* pGlobalContext = ScriptingCore::getInstance()->getGlobalContext();

// Create a GameInfo object first
game::info::GameInfo gameInfo;

// add RoleInfo object to repeated field
game::info::RoleInfo *pRoleInfo1 = gameInfo.add_roleinfo();
pRoleInfo1->set_name("cpp_name1");
pRoleInfo1->set_type(game::enumeration::FIGHTER);

// add another RoleInfo object to repeated field
game::info::RoleInfo *pRoleInfo2 = gameInfo.add_roleinfo();
pRoleInfo2->set_name("cpp_name2");
pRoleInfo2->set_type(game::enumeration::BOWMAN);

// add ItemInfo object to repeated field
game::info::ItemInfo *pItemInfo1 = gameInfo.add_iteminfo();
pItemInfo1->set_name("cpp_item1");
pItemInfo1->set_price(100);

// add another ItemInfo object to repeated field
game::info::ItemInfo *pItemInfo2 = gameInfo.add_iteminfo();
pItemInfo2->set_name("cpp_item2");
pItemInfo2->set_price(200);

// SerializeToString
std::string gameInfoString;
gameInfo.SerializeToString(&gameInfoString);

// convert gameInfoString to jsval to send to javascript
JSAutoCompartment ac(pGlobalContext, pGlobalObject);
int length = gameInfoString.length();
JSObject* pMessageJSObject = JS_NewArrayBuffer(pGlobalContext, length);
uint8_t* pMessageData = JS_GetArrayBufferData(pMessageJSObject);
memcpy(
    (void*)pMessageData, 
    (const void*)gameInfoString.c_str(), 
    gameInfoString.length()
);

JS::RootedValue ret(pGlobalContext);
jsval arg = OBJECT_TO_JSVAL(pMessageJSObject);
ScriptingCore::getInstance()->executeFunctionWithOwner(
    OBJECT_TO_JSVAL(pGlobalObject), "test_protobuf", 1, &arg, &ret
);
```

<br />

#### Receive from `C++`

```javascript
// cc_js_bridge.js
test_protobuf = function (data) {
    var ProtoBuf = dcodeIO.ProtoBuf;
    var GameInfoBuilder = ProtoBuf.newBuilder();
    
    // load .proto
    ProtoBuf.protoFromContents(
        "res/GameEnum.proto", 
        GameInfoBuilder, 
        jsb.fileUtils.getStringFromFile("res/GameEnum.proto")
    );
    ProtoBuf.protoFromContents(
        "res/GameInfo.proto", 
        GameInfoBuilder, 
        jsb.fileUtils.getStringFromFile("res/GameInfo.proto")
    );
    
    // decode protobuf message from C++ 
    var Game = GameInfoBuilder.build("game");
    var gameInfo = Game.info.GameInfo.decode(data, "utf8");
    // ...
}
```

Add a `protoFromContents` `function` to `protobuf.js` :

``` javascript
ProtoBuf.protoFromContents = function(filename, builder, contents) {
    return contents === null ? null : ProtoBuf.loadProto(
        contents, builder, filename
    );
};
```

<br />

#### Send from `javascript`

```javascript
var gameInfoOut = new Game.info.GameInfo();
    
var roleInfo1 = new Game.info.RoleInfo();
roleInfo1.name = "js_name1";
roleInfo1.type = Game.enumeration.RoleType.FIGHTER;
gameInfoOut.roleInfo.push(roleInfo1);

var roleInfo2 = new Game.info.RoleInfo();
roleInfo2.name = "js_name2";
roleInfo2.type = Game.enumeration.RoleType.BOWMAN;
gameInfoOut.roleInfo.push(roleInfo2);

var itemInfo1 = new Game.info.ItemInfo();
itemInfo1.name = "js_item1";
itemInfo1.price = 100;
gameInfoOut.itemInfo.push(itemInfo1);

var itemInfo2 = new Game.info.ItemInfo();
itemInfo2.name = "js_item2";
itemInfo2.price = 200;
gameInfoOut.itemInfo.push(itemInfo2);

return gameInfoOut.encodeAB();
```

<br />

#### Receive from `javascript`

```C++
// ...
JS::RootedValue ret(pGlobalContext);
jsval arg = OBJECT_TO_JSVAL(pMessageJSObject);
ScriptingCore::getInstance()->executeFunctionWithOwner(
    OBJECT_TO_JSVAL(pGlobalObject), "test_protobuf", 1, &arg, &ret
);
// convert JS::RootedValue ret to protobuf message
game::info::GameInfo* pGameInfo = new game::info::GameInfo;
JSObject* pRetJSObject = ret.toObjectOrNull();
if(pRetJSObject && JS_IsArrayBufferObject(pRetJSObject)) {
    uint8_t* pDataBuffer = nullptr;
    int dataBufferCount = 0;
    pDataBuffer = JS_GetArrayBufferData(pRetJSObject);
    dataBufferCount = JS_GetArrayBufferByteLength(pRetJSObject);
    pGameInfo->ParseFromArray(pDataBuffer, dataBufferCount);
    // ...
}
```

<br />

## Others

### Sublime Text3 `.proto` plugin

[Protocol Buffer Syntax](https://packagecontrol.io/packages/Protocol%20Buffer%20Syntax)



>   https://github.com/google/protobuf
>
>   https://github.com/google/protobuf/releases