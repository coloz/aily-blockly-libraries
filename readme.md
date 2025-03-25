# aily blockly库开发 

## 库结构
aily blockly库基本遵循google blockly库结构，使用npm包管理形式管理库的版本及相关必要信息。一个aily blockly库的结构如下：
```
library-name  
 |- block.json             // aily blockly block文件
 |- generator.js           // aily blockly generator文件
 |- toolbox.json           // aily blockly toolbox文件
 |- package.json           // npm包管理文件
 |- src.7z                 // Arduino库源文件，请使用7z极限压缩后放在库中
 |- readme.md              // 说明文件，如果使用了开源库，请进行说明
 |- examples               // 库示例程序
     |- ex1.json
     |- ex2.json
```

## block.json
aily blockly json在原始的blockly json上进行了扩展。这里以servo lib（舵机库）为例进行说明，舵机库json如下：  
```json
[
    {
        "inputsInline": true,
        "message0": "舵机%1移动到%2",
        "type": "servo_write",
        "args0": [
            {
                "type": "field_variable",
                "name": "OBJECT",
                "variable": "servo1",
                "variableTypes": [
                    "Servo"
                ],
                "defaultType": "Servo"
            },
            {
                "type": "field_number",
                "name": "ANGLE",
                "value": 0
            }
        ],
        "previousStatement": null,
        "nextStatement": null,
        "colour": "#03a9f4"
    },
    {
        "inputsInline": true,
        "message0": "舵机%1连接到引脚%2",
        "type": "servo_attach",
        "args0": [
            {
                "type": "field_variable",
                "name": "OBJECT",
                "variable": "servo1",
                "variableTypes": [
                    "Servo"
                ],
                "defaultType": "Servo"
            },
            {
                "type": "field_dropdown",
                "name": "PIN1",
                "options": "${board.digitalPins}"
            }
        ],
        "previousStatement": null,
        "nextStatement": null,
        "colour": "#03a9f4"
    }
]
```

### block配置  
json文件中`blocks数组`即是具体的block配置，可以参照[blockly官方文档](https://developers.google.com/blockly/guides/create-custom-blocks/define-blocks)了解相关配置。  
单一block示例如下：  

```json
{
    "inputsInline": true,              // block样式，true输入项目为一行显示，flase为多行显示
    "message0": "舵机%1连接到引脚%2",   // block信息，定义该block上显示的内容
    "type": "servo_attach",           // block类型，该参数必须是唯一的
    "previousStatement": null,         // 前置连接，配置该block之前的block类型  
    "nextStatement": null,             // 后置连接，配置该block之后的block类型  
    "colour": "#03a9f4",               // block颜色
    "args0": [                         // 输入项
        {
            "type": "field_variable",
            "name": "OBJECT",
            "variable": "servo1",
            "variableTypes": [
                "Servo"
            ],
            "defaultType": "Servo"
        },
        {
            "type": "field_dropdown",
            "name": "PIN1",
            "options": "${board.digitalPins}"
        }
    ]
}
```



## generator.js 
generator.js负责将block转换成代码，每一个block都有一个对应的generator函数。对于Arduino库，generator函数编写形式可参考：
```javascript
Arduino.forBlock['servo_attach'] = function(block, generator) {
  // 实现代码参考blockly官方文档
  return ''
};

Arduino.forBlock['servo_write'] = function(block, generator) {
  // 实现代码参考blockly官方文档
  return ''
};

```

### 添加附加代码
在generator.js中block对应的代码使用retuen返回，对于大多数Arduino出了block对应代码外，还需要添加额外的附加代码。如调用servo库需要添加库引用，并创建一个Servo对象：
```c++
#include <Servo.h>
Servo myservo;
```
此时可在generator.js对应的函数中添加如下语句用以在程序其他位置添加库引用和新建对象语句：
```javascript
Arduino.forBlock['servo_attach'] = function(block, generator) {
  generator.addLibrary('#include <Servo.h>','#include <Servo.h>');
  generator.addVariable('Servo myservo','Servo myservo');
  // 实现代码参考blockly官方文档
  return ''
};
```
可用的函数有：
```javascript
addMacro(tag, code);
addLibrary(tag, code);
addVariable(tag, code);
addObject(tag, code);
addFunction(tag, code);
addSetup(tag, code);
addUserSetup(tag, code);
addUserLoop(tag, code);
addLoop(tag, code);
```
使用以上函数，可以将对应的代码分别加入到程序的以下位置：

```c++
// [宏定义 macro]
#defined PIN 3
// [库引用 library]
#include <Servo.h>
// [变量 variable]
static const int servoPin = 4;
// [对象 object]
Servo servo1;
// [函数 function]

void setup() {
// [初始化部分 setup]
    Serial.begin(115200);
// [用户自定义 userSetup]
    servo1.attach(servoPin);
}

void loop() {
   
// [主程序 loop]
    for(int posDegrees = 0; posDegrees <= 180; posDegrees++) {
        servo1.write(posDegrees);
        Serial.println(posDegrees);
        delay(20);
    }

    for(int posDegrees = 180; posDegrees >= 0; posDegrees--) {
        servo1.write(posDegrees);
        Serial.println(posDegrees);
        delay(20);
    }
}
```

### 注意事项
如果在generator.js需要创建函数，请不要创建全局函数，使用如下方式将函数定义到Arduino中，如：
```js
Arduino.newFunction = function(value) {
  return 'ok';
};
```

## toolbox.json 
Toolbox是呈现在blockly界面左侧的block菜单，示例如下：
```json
{
    "kind": "category",
    "name": "Servo",
    "contents": [
        {
            "kind": "block",
            "type": "servo_attach"
        },
        {
            "kind": "block",
            "type": "servo_write"
        }
    ]
}
```

## package.json
版本控制文件，采用npm包管理,示例如下：
```json
{
    "name": "@aily-project/lib-servo",  // 库的包名以 @aily-project/lib- 开头
    "nickname":"舵机驱动库",             // 用于在库管理器中显示的名称
    "author": "奈何col",
    "description": "舵机控制支持库，支持Arduino UNO、MEGA、ESP8266、ESP32等开发板", // 简短的介绍，不超过50字
    "version": "0.0.1",
    "compatibility": {                  // 兼容性说明，支持的核心和电压
        "core": [
            "arduino:avr",
            "esp32:esp32",
            "esp8266:esp8266"
        ],
        "voltage": [3.3,5]
    },
    "keywords": [                       // keywords可以辅助搜索库，建议其中添加分类名、函数名等
        "aily",
        "blockly",
        "servo",
        "servo_attach",
        "servo_write",
    ],
    "scripts": {},
    "dependencies": {},                // 该库依赖了其他库
    "devDependencies": {}
}
```

## 优化/简化方案
在没有歧义的前提下，尽可能简化库的调用，让用户通过尽量少的block使用相关功能，如：
1. 一些引脚功能需要初始化，如调用`digitalWrite(5,HIGH)`时，对应的初始化则为`pinMode(5,OUTPUT)`。推荐block库在调用digitalWrite、digitalRead相关块时，可以自动向程序setup部分添加对应的pinMode代码。
2. 一些库需要初始化，如servo库，需要使用`myservo.attach(9)`，指定舵机连接的引脚，然后再使用`myservo.write(val)`控制舵机转动角度。这个库建议在保留原本的attach、write块的同时，再提供一个简化的block，允许用户直接指定某引脚上的舵机转动到多少角度。
3. 当Arduino库函数的参数为回调函数时，如onebutton库中`button.attachDoubleClick(doubleClick)`,doubleClick为回调函数，则应该创建一个主体为doubleClick函数block，并在generator.js中向程序的setup部分中添加代码`button.attachDoubleClick(doubleClick)`，向程序loop部分添加`button.tick()`.


