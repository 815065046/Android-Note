��Mavenһ����Gradleֻ���ṩ�˹�����Ŀ��һ����ܣ����������õ���Plugin��Gradle��Ĭ�������Ϊ�����ṩ����ೣ�õ�Plugin�����а����й���Java��Ŀ��Plugin������War��Ear�ȡ���Maven��ͬ���ǣ�Gradle���ṩ�ڽ�����Ŀ�������ڹ���ֻ��java Plugin��Project����������Task����ЩTask����ִ�У�Ϊ����Ӫ����һ����ͬMaven����Ŀ��������rty������ض���operty������ض��Ĳ�����

�������Ƕ���̸����������ƣ�Gradle��������������Ҫ��Project��Task��ProjectΪTask�ṩ��ִ�������ģ����е�PluginҪô��Project������������õ�Property��Ҫô��Project����Ӳ�ͬ��Task��һ��Task��ʾһ���߼��Ͻ�Ϊ������ִ�й��̣��������JavaԴ���룬�����ļ������Jar�ļ�������������ִ��һ��ϵͳ������ߵ���Ant�����⣬һ��Task���Զ�ȡ������Project��Property������ض��Ĳ�����


����������һ����򵥵�Task������һ��build.gradle�ļ����������£�

task helloWorld << {
   println "Hello World!"
}

����build.gradle��ͬ��Ŀ¼��ִ�У�

gradle helloWorld

�����

:helloWorld
Hello World!

BUILD SUCCESSFUL

Total time: 2.544 secs

Gradle��Ĭ�������Ϊ�����ṩ�˼������õ�Task������鿴Project��Properties����ʾ��ǰProject�ж��������Task�ȡ�����ͨ��һ������鿴Project�����е�Task��

gradle tasks

