title: Markdown sample #1

# 概要
マークダウンの中にPlantUMLの図を組み込んで、PDF化してみます。
また、.docx ファイルへの変換も試みます。

# PlantUML の組み込み

```PlantUML
@startuml
/'
class "MyClass" as A1
{
    - int : attribute 
    + method(arg:int) : int
}
'/

'class "YourClass" as B1

activate A1
A1 -> B1 : hello
note left : 最初の挨拶
activate B1
A1 <- B1 : hi!
deactivate B1

note right : 最初の挨拶

A1 -> B1 : who is it ?
activate B1
A1 <- B1 : Ι am Bob.
B1 -> B1 : I think it should be rude.
activate B1

A1 <- B1 : How old are you.
deactivate B1
deactivate A1
@enduml
```
