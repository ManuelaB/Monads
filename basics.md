## Scala Basics 
### Functor
The Option type in scala is used to wrap a value that could be null. You can compare
this type with _Nullables_ in Java. To check the value of a nullable type, you often see
nested ifs like the following:
```Java
String housenumber = null;
User user = new User();
if(user != null){
  Adress adress = user.getAdress();
  if(adress != null){
    housenumber = adress.getHousenumber();
  }
}
```
