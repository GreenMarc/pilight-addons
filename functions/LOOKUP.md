# LOOKUP function

## Introduction
The LOOKUP function has the following format:
```
LOOKUP(haystack, needle, default)
```
and is intended to fetch values from lists of key=value pairs separated by "&" such as 
```
LOOKUP(on=aan&off=uit, mydevice.state) ......
```
Such a list can be a fixed string as shown above, or a device variable. The needle can be a number, string or a device variable. 

The LOOKUP function is most useful when used with webswitch devices returning multiple parameter values, but it can also help to simplify your rules.
## Usage
### Translation of device variables
Let's say we want to display the state of one of our devices in a label in our own language. As an example, we could have in our config:
```
                 "mylabel": {
                        "protocol": [ "generic_label" ],
                        "id": [{
                                "id": 1
                        }],
                        "label": ""
                        "color": "black"
                    "mylamp": {
                        "protocol": [ "kaku_switch" ],
                        "id": [{
                                "id": 12345678,
                                "unit": 0
                        }],
                        "state": "off"
                },
```
We wish to show the state of mylamp translated into our own language in mylabel.
Of course we could create separate rules for different states to achieve that, but using LOOKUP, it can simply be done in one rule:
```
"IF mylamp.state IS on OR mylamp.state IS off THEN label DEVICE mylabel TO LOOKUP(on=aan&off=uit, mylamp.state)";
```
We can use (dummy) LABEL devices to store such lists if they are used by multiple rules. Simply  add a label to your config like:
```
                "translate": {
                        "protocol": [ "generic_label" ],
                        "id": [{
                                "id": 2
                        }],
                        "label": "on=aan&off=uit&running=aan&stopped=uit&Red=Rood&Orange=Oranje&Yellow=Geel"
                        "color": "black"
                },
```
and do
```
"IF mylamp.state IS on OR mylamp.state IS off THEN label DEVICE mylabel TO LOOKUP(translate.label, mylamp.state)";
```
*** Handling missing variables
The LOOKUP function takes an optional third parameter. This parameter can be a string, a number, a single asterisk (*), or a single dollar sign ($). The string or number provided will be returned if the key searched for doesn't exist.  
If an asterisk is entered, the key itself will be returned in that case and with a dollar sign the whole "haystack" will be returned.

Without the third parameter, the string "?" will be returned if the key is not found.

** Value types
The type of the value returned by the LOOKUP function can be either a number, or a string, depending on its value.
Example:
Let's assume that the value of label1.label "a=1&b=two"
To use the value of "a" in a rule we can do:
```
IF ... LOOKUP(mydevice.option, a) == 1 THEN ...
```
We need to use ==, because the value of "a" is numeric.
To use the value of "b":
```
IF .... LOOKUP(mydevice.option, b) IS two THEN ...
```
Here we need to use IS (or ISNOT) because the value of "b" is a string.
If you are using the result of the LOOKUP function with an operator, it is advised to set a value of the type matching the operator as third parameter. 
So, if c is supposed to have a numeric value but doesn't exist, an error will occur if you do:
```
 IF ... LOOKUP(mydevice.option, c) != 0 THEN ...
```
but not if you do:
```
 IF ... LOOKUP(mydevice.option, c, 0) != 0 THEN ...
```
In general you should choose a default value that normally will not be returned.

### Storing and retrieving own variables danamically
Because the key=vakue pairs can be stored in generic label devices, this also gives us a simple way of storing (and retrieving) variables dynamically by writing key=value pairs to a label device:
```
IF ... THEN label DEVICE mylabel TO state= somedevice.value & id= somedevice.id.....
```
**N.B. Spaces are required here to separate the device values from the = and & signs. These spaces are ignored by the LOOKUP function.**
You then can use the LOOKUP funtion to retrieve and use the values in your rules.

**Some more advanced examples

Devices
```
                "bewatering_status": {
                        "protocol": [ "webswitch" ],
                        "id": [{
                                "id": 4
                        }],
                        "method": "GET",
                        "on_uri": "http://192.168.1.13/gardena/",
                        "off_uri": "http://192.168.1.13/gardena/",
                        "on_query": "c=status",
                        "off_query": "c=status",
                        "on_success": "a=b",
                        "off_success": "a=b",
                        "response": "timestamp=2016-05-16 10:29&besturing=pilight&bewatering=off&flow=off&soil=moist&temp=10&humi=52",
                        "state": "stopped"
                },
                                "statuslabel": {
                        "protocol": [ "generic_label" ],
                        "id": [{
                                "id": 21
                        }],
                        "label": "[ 2016-05-16 10:29 ] - Besturing: pilight - Bewatering: uit - Flow: uit - Bodem: vochtig - Temp 10 gr - Luchtvochtigheid: 52 %",
                        "color": "black"
                },
  ```

Rules:
```
 
```