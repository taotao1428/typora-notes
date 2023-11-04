# maven ant插件

antrun插件利用apache ant可以实现很多功能，例如拷贝，解压和压缩等操作



maven插件配置，将external文件夹内容压缩

```xml
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-antrun-plugin</artifactId>
                <version>1.8</version>
                <executions>
                    <execution>
                        <id>run-tar</id>
                        <goals>
                            <goal>run</goal>
                        </goals>
                        <phase>validate</phase>
                        <configuration>
                            <target>
                                <tar destfile="${basedir}/target/external.tar.gz" compression="gzip" basedir="${basedir}/external"/>
                            </target>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
```



<img src="/java虚拟机/.assert/maven-ant插件/image-20230909125823684.png" alt="image-20230909125823684" style="zoom:50%;" />



<img src="/java虚拟机/.assert/maven-ant插件/image-20230909125858857.png" alt="image-20230909125858857" style="zoom:50%;" />



## apache ant官网

https://ant.apache.org

功能介绍

https://ant.apache.org/manual-1.9.x/index.html

![image-20230909125517105](/java虚拟机/.assert/maven-ant插件/image-20230909125517105.png)



## 打包插件

maven-assembly-plugin