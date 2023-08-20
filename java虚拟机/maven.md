

# maven



Maven 有生命周期和phase



![在这里插入图片描述](/java虚拟机/.assert/maven/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Fpbnd1eGlhbjE5ODkxMjEx,size_16,color_FFFFFF,t_70.png)



## maven插件

1. maven插件可以包含多个goal，每个goal对应一个操作
2. 每个goal可以配置一个默认执行phase。当执行该phase时，该goal会自动执行
3. 第三方插件需要手动配置execution才可以在对应的phase执行goal。



## 配置execution



```xml
  <plugin>
      <groupId>org.fusesource.hawtjni</groupId>
      <artifactId>maven-hawtjni-plugin</artifactId>
      <version>1.15</version>

      <configuration>
          <nativeSourceDirectory>${nativeSourceDirectory}</nativeSourceDirectory>
      </configuration>
      <executions>
        
        <execution>
              <id>build-native-lib</id>
              <configuration>
                  <name>native-math</name>
                  <nativeSourceDirectory>${nativeSourceDirectory}</nativeSourceDirectory>
              </configuration>
              <!-- 如果不配置phase，每个goal将会分别按照goal默认的phase执行 -->
              <goals>
                  <goal>generate</goal>
                  <goal>build</goal>
                  <goal>package-source</goal>
              </goals>
          </execution>
        
          <execution>
              <id>build-native-lib</id>
              <!-- 配置phase -->
              <phase>package</phase>
              <configuration>
                  <name>native-math</name>
                  <nativeSourceDirectory>${nativeSourceDirectory}</nativeSourceDirectory>
              </configuration>
             <!-- 如果配置phase，每个goal将会在该phase执行 -->
              <goals>
                  <goal>generate</goal>
                  <goal>build</goal>
              </goals>
          </execution>
      </executions>
  </plugin>
```



## 执行命令

```
# 


# 按照plugin:goal执行，相当于直接执行某个插件的goal
mvn clean:clean
```

### 按照phase执行插件

按照phase执行插件，phase同时会执行同一生命周期内前面的phase。例如执行package，会自动执行validate，compile，test，packagee。每个phase对应的plugin:goal会自动执行。

```
mvn [phase]
```



<img src="/java虚拟机/.assert/maven/image-20230821001026588.png" alt="image-20230821001026588" style="zoom:50%;" />



### 按照plugin:goal执行

按照plugin:goal执行，相当于直接执行某个插件的goal

```
mvn clean:clean
```

![image-20230821001046821](/java虚拟机/.assert/maven/image-20230821001046821.png)