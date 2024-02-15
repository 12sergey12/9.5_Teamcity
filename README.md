### Домашнее задание к занятию 11 «Teamcity» Баранов Сергей

### Подготовка к выполнению

1. В Yandex Cloud создайте новый инстанс (4CPU4RAM) на основе образа `jetbrains/teamcity-server`.

2. Дождитесь запуска teamcity, выполните первоначальную настройку.

3. Создайте ещё один инстанс (2CPU4RAM) на основе образа `jetbrains/teamcity-agent`. Пропишите к нему переменную окружения `SERVER_URL: "http://<teamcity_url>:8111"`.

4. Авторизуйте агент.

5. Сделайте fork [репозитория](https://github.com/aragastmatb/example-teamcity).

6. Создайте VM (2CPU4RAM) и запустите [playbook](./infrastructure).

![monitoring](https://github.com/12sergey12/9.5_Teamcity/blob/main/images/9.5-yandex_cloud.png)

## Основная часть

1. Создайте новый проект в teamcity на основе fork.

2. Сделайте autodetect конфигурации.

3. Сохраните необходимые шаги, запустите первую сборку master.

![monitoring](https://github.com/12sergey12/9.5_Teamcity/blob/main/images/9.5-3.png)

4. Поменяйте условия сборки: если сборка по ветке `master`, то должен происходит `mvn clean de
ploy`, иначе `mvn clean test`.

![monitoring](https://github.com/12sergey12/9.5_Teamcity/blob/main/images/9.5-4.png)

5. Для deploy будет необходимо загрузить [settings.xml](./teamcity/settings.xml) в набор конфигураций maven у teamcity, предварительно записав туда креды для подключения к nexus.

6. В pom.xml необходимо поменять ссылки на репозиторий и nexus.

pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	
	<groupId>org.netology</groupId>
	<artifactId>plaindoll</artifactId>
	<packaging>jar</packaging>
	<version>0.0.2</version>

	<properties>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
	</properties>
	<distributionManagement>
		<repository>
				<id>nexus</id>
				<url>http://158.160.35.78:8081/repository/maven-releases</url>
		</repository>
	</distributionManagement>
	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-shade-plugin</artifactId>
				<version>3.2.4</version>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>shade</goal>
						</goals>
						<configuration>
							<transformers>
								<transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
									<mainClass>plaindoll.HelloPlayer</mainClass>
								</transformer>
							</transformers>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>

</project>
```

7. Запустите сборку по master, убедитесь, что всё прошло успешно и артефакт появился в nexus.

![monitoring](https://github.com/12sergey12/9.5_Teamcity/blob/main/images/9.5-7.png)

8. Мигрируйте `build configuration` в репозиторий.

![monitoring](https://github.com/12sergey12/9.5_Teamcity/blob/main/images/9.5-8.1.png)

![monitoring](https://github.com/12sergey12/9.5_Teamcity/blob/main/images/9.5-8.2.png)

[.teamcity](https://github.com/12sergey12/example-teamcity/tree/master/.teamcity)

9. Создайте отдельную ветку `feature/add_reply` в репозитории.

10. Напишите новый метод для класса Welcomer: метод должен возвращать произвольную реплику, содержащую слово `hunter`.

```
package plaindoll;

public class Welcomer{
	public String sayWelcome() {
		return "Welcome home, good hunter. What is it your desire?";
	}
	public String sayFarewell() {
		return "Farewell, good hunter. May you find your worth in waking world.";
	}
	public String sayNeedGold(){
		return "Not enough gold";
	}
	public String saySome(){
		return "something in the way";
	}
	public String sayHunter() {
		return "Please take care of yoursalf, hunter.";
  }
}
```

11. Дополните тест для нового метода на поиск слова `hunter` в новой реплике.

```
package plaindoll;

import static org.hamcrest.CoreMatchers.containsString;
import static org.junit.Assert.*;

import org.junit.Test;

public class WelcomerTest {
	
	private Welcomer welcomer = new Welcomer();

	@Test
	public void welcomerSaysWelcome() {
		assertThat(welcomer.sayWelcome(), containsString("Welcome"));
	}
	@Test
	public void welcomerSaysFarewell() {
		assertThat(welcomer.sayFarewell(), containsString("Farewell"));
	}
	@Test
	public void welcomerSaysHunter() {
		assertThat(welcomer.sayWelcome(), containsString("hunter"));
		assertThat(welcomer.sayFarewell(), containsString("hunter"));
	}
	@Test
	public void welcomerSaysSilver(){
		assertThat(welcomer.sayNeedGold(), containsString("gold"));
	}
	@Test
	public void welcomerSaysSomething(){
		assertThat(welcomer.saySome(), containsString("something"));
	}
	@Test
	public void welcomernetSaysHunter(){
		assertThat(welcomer.sayHunter(), containsString("hunter"));
	}
}
```

12. Сделайте push всех изменений в новую ветку репозитория.

13. Убедитесь, что сборка самостоятельно запустилась, тесты прошли успешно.

![monitoring](https://github.com/12sergey12/9.5_Teamcity/blob/main/images/9.5-13.png)

14. Внесите изменения из произвольной ветки `feature/add_reply` в `master` через `Merge`.

15. Убедитесь, что нет собранного артефакта в сборке по ветке `master`.

16. Настройте конфигурацию так, чтобы она собирала `.jar` в артефакты сборки.

Для того, чтобы добавить артефакты в сборку нужно изменить свойства проекта Artifact paths в разделе General Settings, где вписать нуобходимые для включения файлы (можно использовать маскирование, например: target/*.jar)

17. Проведите повторную сборку мастера, убедитесь, что сбора прошла успешно и артефакты собраны.

![monitoring](https://github.com/12sergey12/9.5_Teamcity/blob/main/images/9.5-17.png)

18. Проверьте, что конфигурация в репозитории содержит все настройки конфигурации из teamcity.

![monitoring](https://github.com/12sergey12/9.5_Teamcity/blob/main/images/9.5-18.1.png)

[Netology_ExampleTeamcity_Build.xml](https://github.com/12sergey12/example-teamcity/blob/master/.teamcity/Netology_ExampleTeamcity/buildTypes/Netology_ExampleTeamcity_Build.xml)

```
<?xml version="1.0" encoding="UTF-8"?>
<build-type xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" uuid="89c7e65e-f122-4038-b567-a96e0a2ce3e5" xsi:noNamespaceSchemaLocation="https://www.jetbrains.com/teamcity/schemas/2021.1/project-config.xsd">
  <name>Build</name>
  <description />
  <settings>
    <options>
      <option name="artifactRules" value="target/*.jar" />
    </options>
    <parameters />
    <build-runners>
      <runner id="Maven2" name="mnv clean deploy" type="Maven2">
        <conditions>
          <contains name="teamcity.build.branch" value="master" />
        </conditions>
        <parameters>
          <param name="goals" value="clean deploy" />
          <param name="localRepoScope" value="agent" />
          <param name="maven.path" value="%teamcity.tool.maven.DEFAULT%" />
          <param name="pomLocation" value="pom.xml" />
          <param name="runnerArgs" value="-Dmaven.test.failure.ignore=true" />
          <param name="teamcity.step.mode" value="default" />
          <param name="userSettingsSelection" value="settings.xml" />
        </parameters>
      </runner>
      <runner id="mnv_clean_test" name="mnv clean test" type="Maven2">
        <conditions>
          <does-not-contain name="teamcity.build.branch" value="master" />
        </conditions>
        <parameters>
          <param name="goals" value="clean test" />
          <param name="localRepoScope" value="agent" />
          <param name="maven.path" value="%teamcity.tool.maven.DEFAULT%" />
          <param name="pomLocation" value="pom.xml" />
          <param name="teamcity.coverage.emma.include.source" value="true" />
          <param name="teamcity.coverage.emma.instr.parameters" value="-ix -*Test*" />
          <param name="teamcity.coverage.idea.includePatterns" value="*" />
          <param name="teamcity.coverage.jacoco.patterns" value="+:*" />
          <param name="teamcity.step.mode" value="default" />
          <param name="teamcity.tool.jacoco" value="%teamcity.tool.jacoco.DEFAULT%" />
          <param name="userSettingsSelection" value="userSettingsSelection:default" />
        </parameters>
      </runner>
    </build-runners>
    <vcs-settings>
      <vcs-entry-ref root-id="Netology_ExampleTeamcity_HttpsGithubCom12sergey12exampleTeamcityGitRefsHeadsMaster" />
    </vcs-settings>
    <requirements />
    <build-triggers>
      <build-trigger id="TRIGGER_1" type="vcsTrigger">
        <parameters>
          <param name="branchFilter" value="+:*" />
          <param name="enableQueueOptimization" value="true" />
          <param name="quietPeriodMode" value="DO_NOT_USE" />
        </parameters>
      </build-trigger>
    </build-triggers>
    <build-extensions>
      <extension id="perfmon" type="perfmon">
        <parameters>
          <param name="teamcity.perfmon.feature.enabled" value="true" />
        </parameters>
      </extension>
    </build-extensions>
    <cleanup />
  </settings>
</build-type>
```

19. В ответе пришлите ссылку на репозиторий.

[example-teamcity](https://github.com/12sergey12/example-teamcity)

---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---
