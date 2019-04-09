# SpringCloudNote

## mvn打包后运行

mvn clean package  -Dmaven.test.skip=true

java -jar target/eurekademo-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer1