# XML webszolgáltatások

## Célkitűzés

A gyakorlat célja, hogy a hallgatók megismerjék az XML webszolgáltatások alapjait. A főbb témák: 

- XML webszolgáltatás fejlesztése Spring Boot és Apache CXF segítségével
- XML webszolgáltatás kliens generálása WSDL-ből
- SOAP kérések monitorozása

## Előfeltételek

A labor elvégzéséhez szükséges eszközök:

- Legalább 11-es JDK, pl. OpenJDK: https://adoptopenjdk.net/?variant=openjdk11&jvmVariant=hotspot
- Tetszőleges Java alapú, Mavennel integrálódó IDE. A gyakorlatokhoz kapcsolódó videón a Spring Tools 4 for Eclipse-et használjuk: https://spring.io/tools

## Elméleti alapok

Az XML webszolgáltatások olyan hálózati komponensek vagy alkalmazások, amelyek XML dokumentumok formájában, a SOAP protokoll és a WSDL interfészleíró nyelv segítségével hívhatók meg. Használatuk a 2000-es évek elején terjedt el, és egy időben nagyon népszerűek voltak, ha több platformról meghívható módon akartunk üzleti funkciókat hálózaton elérhetővé tenni. Minden fontosabb platformon megjelentek azok a web service toolkitek, melyek támogatják az XML webszolgáltatások fejlesztését, illetve a WSDL fájl alapján a webszolgáltatást meghívó kliens oldali kód generálását. Ezek használatával elfedhetők az XML, a SOAP, a HTTP alacsony szintű részletei, szerver oldalon néhány sor kóddal publikálható egy üzleti logikai metódus, kliens oldalon pedig egyszerű metódushívásként látszik a web service hívás. 

Manapság sok helyen JSON alapú RESTful webszolgáltatások, vagy más kommunikációs formák vették át az XML webszolgáltatások helyét, de sok nagyvállalati rendszerben továbbra is megtalálhatók.

A Java Enterprise Edition kezdetben a JAX-RPC API-t vezette be az XML webszolgáltatások magas szintű támogatására. Ennek 2.0-s verziója már a JAX-WS (Java API for XML Web Services) nevet kapta, és a mai napig ez a standard API Java világban az XML webszolgáltatásokhoz. Mint minden Java EE API, ez is csak interfészeket, annotációkat definiál, és több implementációja létezik. Mi ezek közül az Apache CXF-et fogjuk használni. Érdekesség, hogy a CXF a JAX-RS API-t is megvalósítja, amivel (akár JSON alapú) RESTful webszolgáltatásokat tudunk fejleszteni. A CXF-et Spring Boot környezetben fogjuk használni, a szolgáltatás publikálása így spring-specifikus configuration osztály megírását fogja igényelni. Java EE alkalmazásszerver használata esetén a JAX-WS annotációkkal megjelölt osztályokat automatikusan felismeri a szerver, így ott még kevesebb kódra lenne szükség. 

A JAX-WS API a Java 6-ban "leszivárgott" a Java SE-be is, így lehetségessé vált akár Java SE alkalmazásból futtatni vagy meghívni egy webszolgáltatást. A Java 11-ben ezt már kivették, innentől kezdve külön függőségként kell behúzni.

## Feladat 1: Eclipse indítása
1. Indítsuk el az Eclipse-et innen: `C:\Work\javaee\eclipse\eclipse.exe`. (Fontos, hogy lehet egy `D:\eclipse` mappa is, nekünk _nem_ az kell.) Otthoni megoldás esetén természetesen a saját IDE-t indítsd el.
2. Indításkor megkérdezi, hova akarunk dolgozni (workspace), itt a laborbeli megoldás esetében válasszuk ezt: `C:\Work\javaee\workspaces\hatteralk`
3. Ha az indulás után a Project Explorer-ben ott van egy korábbi gyakorlatról bármilyen projekt, azt töröljük ki: a projekten jobb klikk / _Delete_, amikor rákérdez, pipáljuk be, hogy a fájlrendszerről is törlődjön.

## Feladat 2: Projekt létrehozása

1. File > New > Spring Starter Project
2. A varázslóban adjunk nevet a projektnek (pl. wslab), töltsük ki a group, artifact és package mezőket (pl. hu.bme.aut.hatter, wslab, hu.bme.aut.hatter.wslab). A Type legyen Maven, a Java Version 11.
3. Next után a függőségek közt válasszuk ki a Spring Web-et



## Feladat 3: Valutaváltó üzleti logika elkészítése

1. Hozzunk létre egy új osztályt CurrencyPair néven, amely két Stringet tartalmaz, source és target néven. (Miről mire akarunk átváltani.)
2. Generáljunk hozzá gettert, settert, default konstruktort, a tagváltozókat inicializáló konstruktort, equals és hashCode metódust!
3. Írjunk egy CurrencyService osztályt, amely egy private Map-ben CurrencyPair kulcsokat és Double értékeket tárol, és inicializáljuk is néhány adattal!
4. Írjunk egy getRate metódust, amely String source és String target argumentumokhoz visszaadja az árfolyamot a map segítségével!



## Feladat 4: Valutaváltó szolgáltatás publikálása XML webszolgáltatásként

1. Vegyük fel a pom-ba az alábbi függőséget!

    ```xml
    <dependency>
      <groupId>org.apache.cxf</groupId>
      <artifactId>cxf-spring-boot-starter-jaxws</artifactId>
      <version>3.4.3</version>
    </dependency>
    ```

1. Emeljük ki egy interfészbe a CurrencyService getRate metódusát! (Refactor > Extract Interface...). Legyen a neve pl. ICurrencyService!

1. Tegyük rá az interfészre a @WebService annotációt!

1. Hozzunk létre egy új osztályt WebServiceConfig néven, a WslabApplication osztály package-ében, amelyben regisztráljuk a webservice osztályunkat a CXF-ben. Ez egy springes @Configuration osztály lesz:
    ```java
    @Configuration
    public class WebServiceConfig {

    	@Autowired
    	private Bus bus;
    
    	@Autowired
    	private CurrencyService currencyService;

    	@Bean
    	public Endpoint endpoint() {
        	EndpointImpl endpoint = new EndpointImpl(bus, currencyService);
        	endpoint.publish("/currency");
        	return endpoint;
    	}
    }
    ```
    
1. Figyeljünk oda az importoknál: 

    - Endpoint a javax.xml.ws package-ből
    - EndpointImpl a org.apache.cxf.jaxws package-ből

1. Futtassuk az alkalmazást: Debug as > Spring Boot App

    - Ha portütközés miatt nem indul az alkalmazás, vagy állítsd le azt az alkalmazást, ami a 8080-as porton fut, vagy az src\main\resources\application.properties fájlban adj meg másik portot, majd mentés után futtasd újra, pl. server.port=8081 Fontos, hogy a későbbiekben is ezt használd a leírás további részében by default szereplő 8080 helyett.

1. Böngészőben nyissuk meg és értelmezzük a webszolgáltatás végponján kigenerált WSDL fájlt: http://localhost:8080/services/currency?wsdl

    - A *services* a default CXF url prefix, de át is lehetne konfigolni megfelelő property segítségével
    - Az URI *currency* részét mi határoztuk meg az endpoint.publish hívásnál



## Feladat 5: Kliens fejlesztése az XML webszolgáltatáshoz

1. Hozzunk létre egy új maven projektet: File > New > Project... > Maven Project
   - Create a simple project
   - group id, artifactid pl. hu.bme.aut.hatter, wslab-client
   
1. Az új projekt pom-jában állítsuk be, hogy Java 11-et akarunk használni:
    ```xml
    <properties>
    	<maven.compiler.target>11</maven.compiler.target>
    	<maven.compiler.source>11</maven.compiler.source>
    </properties>
    ```
    
1. Vegyük fel a pom-ba a jaxws-maven-plugint, amellyel kliens oldali kódot generálunk a webszolgáltatás meghívásához:
    ```xml
    <build>
		<plugins>
			<plugin>
				<groupId>org.codehaus.mojo</groupId>
				<artifactId>jaxws-maven-plugin</artifactId>
				<executions>
					<execution>
						<id>generate-reports-ws-code</id>
						<phase>generate-sources</phase>
						<goals>
							<goal>wsimport</goal>
						</goals>
						<configuration>
							<packageName>hu.bme.aut.hatter.wslab.client</packageName>
							<wsdlUrls>
								<wsdlUrl>http://localhost:8080/services/currency?wsdl</wsdlUrl>
							</wsdlUrls>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
    ```
    
1. A pom mentése után szükség lehet arra, hogy a projektet kijelölve Alt+F5-tel frissítsük a projektet.

1. A generált kód a target\generated-sources\wsimport mappában fog megjelenni. Ha ez nem jelenik meg by default Java package-ként, akkor vegyük fel Source Folderként. (Jobb klikk a mappán > Build Path > Use as Source Folder)

1. A generált kódban fordítási hibákat fogunk látni, ennek az az oka, hogy a Java 11-ből kivették a JAX-WS API-t, a JDK méretének csökkentése érdekében. Ez az alábbi függőség felvételével orvosolható a pom-ban:

    ```xml
    <dependencies>
		<dependency>
			<groupId>com.sun.xml.ws</groupId>
			<artifactId>rt</artifactId>
			<version>2.3.3</version>
		</dependency>
    </dependencies>
    ```
1. Tekintsük át a generált kódot!
2. Hozzunk létre egy Main osztályt, main metódussal, amelyben meghívjuk a szolgáltatást!
    ```java
    CurrencyServiceService currencyService = new CurrencyServiceService();
    ICurrencyService currencyServicePort = currencyService.getCurrencyServicePort();
        
    System.out.println(currencyServicePort.getRate("EUR", "HUF"));
    ```



## Feladat 5: SOAP forgalom monitorozása

1. Töltsd le és futtasd az alábbi egyszerű eszközt: [tcpTrace](https://github.com/BMEVIAUBB04/gyakorlat-xml-ws/raw/master/tools/tcpTrace.zip)! Ez egy adott porton figyel, és az oda érkező kéréseket továbbítja egy tetszőleges szerver tetszőleges portjára, miközben megjeleníti a TCP socketen utazó adatokat, mindkét irányban.

2. Add meg, hogy melyik porton figyeljen (pl. 9080), és hova továbbítsa a beérkező kéréseket: Destination Server: localhost, Destination port: 8080

3.  Írjuk át a klienst, hogy ne közvetlen a 8080-as porton futó szerverre küldje a kéréseket, hanem a tcpTrace által nyitott 9080-as portra. Ehhez a currencyServicePort castolása és egy property beállítása szükséges még a webszolgáltatás meghívása előtt:

    ```java
     ((BindingProvider)currencyServicePort).getRequestContext()
            .put(BindingProvider.ENDPOINT_ADDRESS_PROPERTY, "http://localhost:9080/services/currency");
    ```
    
1.  Futtassuk újra az alkalmazást, majd nézzük meg a tcpTrace felületén a kérést és a választ!



## Feladat 6: Repülőjárat kereső szolgáltatás fejlesztése, meghívása és monitorozása (önálló)

1. Fejlessz egy XML webszolgáltatást, amellyel repülőjáratokat lehet keresni indulási és érkezési hely alapján. Egy járatról tudjuk, hogy honnan hova repül, mi az indulási és érkezési időpontja, és hogy mekkora a jegy ára, egy adott pénznemben.
2. A szolgáltatást a valutaváltó projektjében valósítsd meg, de külön végpontként! A repülőjáratokat elegendő a memóriában tárolni, a valutaátváltóhoz hasonlóan. Vegyél fel egy olyan járatot, amelynek indulási helye a te Neptun-kódod és az ár pénzneme EUR!
3. A kliens oldali projekt pom-ját bővítsd ki, hogy az új webszolgáltatáshoz is generáljon osztályokat!
4. A main metódusból hívd meg olyan paraméterekkel a járatkeresőt, hogy a te Neptun-kódodhoz tartozó járat ott legyen a találatok között! Írd ki a talált járatokat, de a pénznem mindegyikben HUF legyen! A pénznem átváltásához a közös feladatban elkészített valutaváltó szolgáltatást hívd meg!
5. Beadandó:
   1. A projekt forráskód zipelve, a target mappa nélkül.
   2. Egy jegyzőkönyv, melyben screenshot szerepel a tcpTrace-ről, miközben a járatkeresést és valutaátváltást végző main metódust futtatod a saját Neptun-kódoddal. Látszódjanak a kérések és válaszok is, készíts több screenshotot, ha szükséges.