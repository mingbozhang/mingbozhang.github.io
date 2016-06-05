---
layout: post
title: "Maven将依赖的所有jar包打成一个jar"
date: 2016-06-04
excerpt: "Maven将依赖的所有jar包打成一个jar"
tags: [maven,java]
---   
## Maven将依赖的所有jar包打成一个jar

有些特殊情况下，需要将多个jar包打包成一个jar文件。如果使用maven可以加入如下插件：   
    
    <build>
		<plugins>
			<plugin>
				<artifactId>maven-assembly-plugin</artifactId>
				<configuration>
					<archive>
						<manifest>
							<!--这里要替换成jar包main方法所在类 -->
							<mainClass>cn.outofmemory.MainClass</mainClass>
						</manifest>
					</archive>
					<descriptorRefs>
						<descriptorRef>jar-with-dependencies</descriptorRef>
					</descriptorRefs>
				</configuration>
				<executions>
					<execution>
						<id>make-assembly</id> <!-- this is used for inheritance merges -->
						<phase>package</phase> <!-- 指定在打包节点执行jar包合并操作 -->
						<goals>
							<goal>single</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>        