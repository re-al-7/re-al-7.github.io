---
title: "Selenium, JUnit y JMeter"
date: 2021-06-25
tags: ["test", "java"]
---


En esta oportunidad vamos a ver como realizar tests con Selenium y exportarlos a JUnit. Y luego leeremos el archivo JAR generado por JUnit para incluirlo en JMeter.


![_config.yml](/images/2021-06-25-Selenium-JMeter.png)


Primero necesitamos instalar algunas cosas:

- [Selenium](https://www.selenium.dev/selenium-ide/)

- [JMeter](https://jmeter.apache.org/download_jmeter.cgi)

- [JMeter Plugin WebDriver](https://jmeter-plugins.org/?search=jpgc-webdriver)

- WebDrivers de [Chrome](https://chromedriver.chromium.org/downloads), [Firefox](https://github.com/mozilla/geckodriver/releases) y/o [Edge](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/)

- [WebDriver Language Bindings](https://www.selenium.dev/downloads/) para Java

- Intellij Idea o algun otro IDE para Java


Con todo esto podemos comenzar a *grabar* nuestras pruebas en Selenium. A continuación tenemos un ejemplo:

![_config.yml](/images/2021-06-25-Selenium-JMeter-Phase-01.png)


Luego podemos exportar el script de Selenium a un archivo JUnit:

![_config.yml](/images/2021-06-25-Selenium-JMeter-Phase-02.png)

![_config.yml](/images/2021-06-25-Selenium-JMeter-Phase-03.png)


Finalmente abrimos nuestro IDE de Java (aqui vamos a usar Intellij Idea) y creamos un proyecto de tipo JAVA para añadir nuestro archivo. También se debe crear una carpeta "lib" con las archivos [WebDriver Language Bindings](https://www.selenium.dev/downloads/) descargados desde la pagina de Selenium. La estructura del proyecto debe quedar mas o menos asi:

![_config.yml](/images/2021-06-25-Selenium-JMeter-Phase-04.png)

Ahora se deben agregar los paquetes (o librerias) necesarias(como el JUnit); pero en caso de las librerías de Selenium, se deben referenciar los JARs. Para ello, dentro de Intellij Idea vamos al menú:

~~~terminal
File &rarr; Project Structure
~~~

Seleccionamos la sección de `Dependencies` en:

~~~terminal
Project Settings &rarr; Project Structure &rarr; Dependencies
~~~

Alli agregamos todos los Jars descargados, incluidos los que se encuentran dentro de la carpeta "libs":

![_config.yml](/images/2021-06-25-Selenium-JMeter-Phase-05.png)


En algunos casos es necesario "ajustar" los selectores de los elementos. También podemos añadir instrucciones para que el test "espere" a la existencia de determinados elementos:

~~~java
WebDriverWait wait = new WebDriverWait(driver, 10);
...
...
...
wait.until(ExpectedConditions.presenceOfElementLocated(By.linkText("test.00325")));
wait.until(ExpectedConditions.elementToBeClickable(By.linkText("test.00325")));
~~~

También se debe configurar el PATH de los WebDrivers; para ello modificamos la función `public void setUp()` para que quede asi:

~~~java
@Before
  public void setUp() {
    System.setProperty("webdriver.chrome.driver","C:\\Program Files\\SeleniumWebDrivers\\chromedriver91.exe");
    System.setProperty("webdriver.gecko.driver","C:\\Program Files\\SeleniumWebDrivers\\geckodriver.exe");
    //driver = new FirefoxDriver();
    driver = new ChromeDriver();
    js = (JavascriptExecutor) driver;
    vars = new HashMap<String, Object>();
  }
~~~

En esta sección se determina si se va a usar Chrome, Firefox o Edge para las pruebas

Para terminar, en el caso de Intellij Idea, es necesario especificar la "configuración" de compilación. Para ello, se va a utilizar un proyecto de NUnit para que quede similar a:


![_config.yml](/images/2021-06-25-Selenium-JMeter-Phase-06.png)


Ya podemos ejecutar el proyecto y ver cómo nuestro *test* grabado en Selenium se reproduce integramente como una caso de pruebas de `JUnit`. Para eso se procede a crear el archivo JAR a partir del código generado. Para ello debemos crear "Artefacts" en Intellij Idea desde el siguiente menu:

~~~terminal
File &rarr; Project Structure
~~~

Seleccionamos la sección de `Artifacts` en:

~~~terminal
Project Settings &rarr; Artifacts &rarr; NEW
~~~


![_config.yml](/images/2021-06-25-Selenium-JMeter-Phase-07.png)


Es importante **marcar** la opción `Include in project builder` y anotar el PATH de salida:


![_config.yml](/images/2021-06-25-Selenium-JMeter-Phase-08.png)


Con todo esto, ya se tiene el archivo JAR necesario para importarlo a `JMeter`. Para ello, copiamos el JAR en el siguiente path:

~~~terminal
\jmeter-x.x\lib\junit
~~~


Ahora, abrimos JMeter y añadimos los siguientes elementos:


~~~terminal
Add &rarr; Threads(Users) &rarr; Thread Group

--Dentro del Thread Group añadir:

Add &rarr; Sampler &rarr; JUnit Request

Add &rarr; Listener &rarr; Summary Report
Add &rarr; Listener &rarr; View Results Tree
Add &rarr; Listener &rarr; Graph Results
Add &rarr; Listener &rarr; Simple Data Writer
~~~


![_config.yml](/images/2021-06-25-Selenium-JMeter-Phase-09.png)

Con esto, podemos ir al modulo `JUnit Request` y marcar la opción `Search for JUnit 4 annotations (instead of JUnit3)`; esto desplegará las ClassName de los JARs que estan en la carpeta `\jmeter-x.x\lib\junit`:

![_config.yml](/images/2021-06-25-Selenium-JMeter-Phase-12.png)

Podemos marcar las opciones `Append assertion errors` y `Append runtime exceptions` que se encuentran en la parte inferior de la ventana de configuración del `JUnit Request`.

Con todo esto, podemos ejecutar el TEST y veremos como se reproduce el script de nuestra librería JUnit, presionando el boton `START` de la barra de herramientas, o presionando los botones `Ctrl + R`

## Bonus
Podemos crear nuestro proyecto como un `WebDriver Sampler`, por lo que debemos descargar el [JMeter Plugin WebDriver](https://jmeter-plugins.org/?search=jpgc-webdriver) y descomprimir el archivo en la carpeta `lib` de la instalación de JMeter.

Ahora, abrimos JMeter y añadimos los siguientes elementos:


~~~terminal
Add &rarr; Config Element &rarr; jp@gc Chrome Driver Config
Add &rarr; Threads(Users) &rarr; Thread Group

--Dentro del Thread Group añadir:

Add &rarr; Sampler &rarr; jp@gc WebDriver Sampler

Add &rarr; Listener &rarr; Summary Report
Add &rarr; Listener &rarr; View Results Tree
Add &rarr; Listener &rarr; Graph Results
Add &rarr; Listener &rarr; Simple Data Writer
~~~

En el elemento `jp@gc WebDriver Sampler` podemos añadir nuestro propio código basado en lo que que se realizó con Selenium y JUnit:

![_config.yml](/images/2021-06-25-Selenium-JMeter-Phase-11.png)

Se puede acceder a [este](https://jmeter-plugins.org/wiki/WebDriverSampler/?utm_source=jmeter&utm_medium=helplink&utm_campaign=WebDriverSampler) enlace para ver la documentación sobre *Web Driver Sampler*.

El codigo es muy similar a lo que se tiene escrito en Java para *JUnit*

~~~java
var pkg = JavaImporter(org.openqa.selenium, org.openqa.selenium.support.ui)
WDS.sampleResult.sampleStart()
try
{
	WDS.browser.get('https://testshop.incognito.ie/')
	var wait = new pkg.WebDriverWait(WDS.browser, 10)
	//Login
	var lnkSigIn = WDS.browser.findElement(pkg.By.linkText("Sign In"));
	lnkSigIn.click();
	var txtUser = WDS.browser.findElement(pkg.By.id("email"));
	txtUser.sendKeys("mailTest20211@example.com");
	var txtPass = WDS.browser.findElement(pkg.By.id("pass"));
	txtPass.sendKeys("Test20211");
	var btnLogin = WDS.browser.findElement(pkg.By.cssSelector(".primary:nth-child(1) > #send2 > span"));
	btnLogin.click();
	
	//GoTo Category
	wait.until(pkg.ExpectedConditions.presenceOfElementLocated(pkg.By.id("ui-id-3")))
	var hrefCategoria = WDS.browser.findElement(pkg.By.id("ui-id-3"));
	hrefCategoria.click();
	
	//Goto Page 5
	wait.until(pkg.ExpectedConditions.presenceOfElementLocated(pkg.By.cssSelector("#layer-product-list > div:nth-child(6) > div.pages > ul > li:nth-child(5) > a > span:nth-child(2)")))
	wait.until(pkg.ExpectedConditions.elementToBeClickable(pkg.By.cssSelector("#layer-product-list > div:nth-child(6) > div.pages > ul > li:nth-child(5) > a > span:nth-child(2)")))
	var hrefPage5 = WDS.browser.findElement(pkg.By.cssSelector("#layer-product-list > div:nth-child(6) > div.pages > ul > li:nth-child(5) > a > span:nth-child(2)"))
	hrefPage5.click()
	
	//Enter product details
	wait.until(pkg.ExpectedConditions.presenceOfElementLocated(pkg.By.linkText("test.00325")))
	wait.until(pkg.ExpectedConditions.elementToBeClickable(pkg.By.linkText("test.00325")))
	var hrefProduct = WDS.browser.findElement(pkg.By.linkText("test.00325"));
	hrefProduct.click();
	
	//Add to cart
	wait.until(pkg.ExpectedConditions.presenceOfElementLocated(pkg.By.cssSelector("#product-addtocart-button > span")))
	wait.until(pkg.ExpectedConditions.elementToBeClickable(pkg.By.cssSelector("#product-addtocart-button > span")))
	var btnAddToCart = WDS.browser.findElement(pkg.By.cssSelector("#product-addtocart-button > span"));
	btnAddToCart.click();
	
	//GoTo Category
	wait.until(pkg.ExpectedConditions.presenceOfElementLocated(pkg.By.id("ui-id-3")))
	hrefCategoria = WDS.browser.findElement(pkg.By.id("ui-id-3"));
	hrefCategoria.click();
	
	//Goto Page 5
	wait.until(pkg.ExpectedConditions.presenceOfElementLocated(pkg.By.cssSelector("#layer-product-list > div:nth-child(6) > div.pages > ul > li:nth-child(5) > a > span:nth-child(2)")))
	wait.until(pkg.ExpectedConditions.elementToBeClickable(pkg.By.cssSelector("#layer-product-list > div:nth-child(6) > div.pages > ul > li:nth-child(5) > a > span:nth-child(2)")))
	hrefPage5 = WDS.browser.findElement(pkg.By.cssSelector("#layer-product-list > div:nth-child(6) > div.pages > ul > li:nth-child(5) > a > span:nth-child(2)"))
	hrefPage5.click()
	
	//Delete cart
	wait.until(pkg.ExpectedConditions.presenceOfElementLocated(pkg.By.cssSelector(".showcart")));
	wait.until(pkg.ExpectedConditions.elementToBeClickable(pkg.By.cssSelector(".showcart")));
	var menuCart = WDS.browser.findElement(pkg.By.cssSelector(".showcart"));
	menuCart.click();
	wait.until(pkg.ExpectedConditions.presenceOfElementLocated(pkg.By.cssSelector(".delete")));
	wait.until(pkg.ExpectedConditions.elementToBeClickable(pkg.By.cssSelector(".delete")));
	var cartDelete = WDS.browser.findElement(pkg.By.cssSelector(".delete"));
	cartDelete.click();
	wait.until(pkg.ExpectedConditions.presenceOfElementLocated(pkg.By.cssSelector(".action-primary > span")));
	wait.until(pkg.ExpectedConditions.elementToBeClickable(pkg.By.cssSelector(".action-primary > span")));
	var confirmDelete = WDS.browser.findElement(pkg.By.cssSelector(".action-primary > span"));
	confirmDelete.click();

	WDS.browser.manage().window().setSize(new pkg.Dimension(1280, 1024))


	//LogOut
	wait.until(pkg.ExpectedConditions.elementToBeClickable(pkg.By.cssSelector("body > div.page-wrapper > div.header-placeholder > div.page-header.page-header-v2 > header > div.header.content > div.header_right > ul > li.customer-welcome > span")));
	var menuUser = WDS.browser.findElement(pkg.By.cssSelector("body > div.page-wrapper > div.header-placeholder > div.page-header.page-header-v2 > header > div.header.content > div.header_right > ul > li.customer-welcome > span"));
	menuUser.click();
	wait.until(pkg.ExpectedConditions.elementToBeClickable(pkg.By.cssSelector(".active .authorization-link > a")));
	var menuLogOut = WDS.browser.findElement(pkg.By.cssSelector(".active .authorization-link > a"));
	menuLogOut.click();
}
catch (err)
{
   WDS.log.error(err.message)
   var screenshot = WDS.browser.getScreenshotAs(pkg.OutputType.FILE)
   screenshot.renameTo(new java.io.File('screenshot.png'))
   exception = err
}
WDS.sampleResult.sampleEnd()
~~~

Finalmente en el apartado de `Chrome Driver Config` se debe establecer el PATH del WebDriver de Chrome:

![_config.yml](/images/2021-06-25-Selenium-JMeter-Phase-10.png)

Al igual que con JUnit, los resultados de la ejecuci;ón se almacenan en los *Listeners*