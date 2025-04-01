Required dependencies: io.ktor

The ContentNegotiation plugin serves two primary purposes: negotiating media types between the client and server and serializing/deserializing the content in a specific format.
The ContentNegotiation plugin serves two primary purposes:

Negotiating media types between the client and server. For this, it uses the Accept and Content-Type headers.
Serializing/deserializing the content in a specific format. Ktor supports the following formats out-of-the-box: JSON, XML, CBOR, and ProtoBuf.
On the client, Ktor provides the ContentNegotiation plugin for serializing/deserializing content.
```
Add dependencies {id="add_dependencies"}
```
ContentNegotiation {id="add_content_negotiation_dependency"}
Note that serializers for specific formats require additional artifacts. For example, kotlinx.serialization requires the ktor-serialization-kotlinx-json dependency for JSON.
```
Serialization {id="serialization_dependency"}
```
Before using kotlinx.serialization converters, you need to add the Kotlin serialization plugin as described in the Setup section.
```
JSON {id="add_json_dependency"}
```
To serialize/deserialize JSON data, you can choose one of the following libraries: kotlinx.serialization, Gson, or Jackson.

Add the ktor-serialization-kotlinx-json artifact in the build script:
```
XML {id="add_xml_dependency"}
```
To serialize/deserialize XML, add the ktor-serialization-kotlinx-xml in the build script:

Note that XML serialization is not supported on jsNode target. {style="note"}
```
CBOR {id="add_cbor_dependency"}
```
To serialize/deserialize CBOR, add the ktor-serialization-kotlinx-cbor in the build script:
```
ProtoBuf {id="add_protobuf_dependency"}
```
To serialize/deserialize ProtoBuf, add the ktor-serialization-kotlinx-protobuf in the build script:
```
Install ContentNegotiation {id="install_plugin"}
Configure a serializer {id="configure_serializer"}
```
Ktor supports the following formats out-of-the-box: JSON, XML, CBOR. You can also implement your own custom serializer.

JSON serializer {id="register_json"}
To register the JSON serializer in your application, call the json method:
```
import io.ktor.server.plugins.contentnegotiation.*
import io.ktor.serialization.kotlinx.json.*

install(ContentNegotiation) {
    json()
}
```
The json method also allows you to adjust serialization settings provided by JsonBuilder, for example:

{src="snippets/json-kotlinx/src/main/kotlin/jsonkotlinx/Application.kt" include-lines="25-30"}

To register the Gson serializer in your application, call the gson method:
```
import io.ktor.server.plugins.contentnegotiation.*
import io.ktor.serialization.gson.*
```

```
install(ContentNegotiation) {
    gson()
}
```
The gson method also allows you to adjust serialization settings provided by GsonBuilder, for example:

{src="snippets/gson/src/main/kotlin/com/example/GsonApplication.kt" include-lines="24-29"}

To register the Jackson serializer in your application, call the jackson method:
```
import io.ktor.server.plugins.contentnegotiation.*
import io.ktor.serialization.jackson.*

install(ContentNegotiation) {
    jackson()
}
```
The jackson method also allows you to adjust serialization settings provided by ObjectMapper, for example:

{src="snippets/jackson/src/main/kotlin/com/example/JacksonApplication.kt" include-lines="26-35"}



```
dependencies {

    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.activity.compose)
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.ui)
    implementation(libs.androidx.ui.graphics)
    implementation(libs.androidx.ui.tooling.preview)
    implementation(libs.androidx.material3)
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)
    androidTestImplementation(platform(libs.androidx.compose.bom))
    androidTestImplementation(libs.androidx.ui.test.junit4)
    debugImplementation(libs.androidx.ui.tooling)
    debugImplementation(libs.androidx.ui.test.manifest)

    implementation(libs.kotlinx.coroutines.android)
    implementation(libs.ktor.server.core)
    implementation(libs.ktor.server.netty)
    implementation(libs.ktor.client.content.negotiation)
    implementation(libs.ktor.serialization.kotlinx.json)
    implementation(libs.coil.compose)
    implementation(libs.coil.network.okhttp)
}
```
```
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
    kotlin("plugin.serialization")
}
```
```
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
    kotlin("plugin.serialization") version "2.1.20" apply false
}
```



ApiService
```
interface ApiService {
    suspend fun fetchRandomDog(): RandomDog
}

class ApiServiceImpl : ApiService {
    private val client = HttpClient(CIO) {
        install(ContentNegotiation) {
            json()
        }
    }

    override suspend fun fetchRandomDog(): RandomDog {
        return client.get("https://random.dog/woof.json").body()
    }
}
```

Dog
```
@Entity(tableName = "dogs")
class Dog {
    @PrimaryKey(autoGenerate = true)
    @NonNull
    @ColumnInfo(name = "id")
    var id: Int = 0
    @ColumnInfo(name = "url")
    var url: String=""
 
    constructor() {}
 
    constructor(url: String) {
        this.url = url
    }
}
```

MainActivity
```
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            SuperAppTheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    Main(
                        modifier = Modifier.padding(innerPadding)
                    )
                }
            }
        }
    }
}

@Composable
fun Main() {
    val navController = rememberNavController()
    Column(Modifier.padding(8.dp)) {
        NavHost(navController, startDestination = NavRoutes.Home.route, modifier = Modifier.weight(1f)) {
            composable(NavRoutes.Home.route) { Home() }
            composable(NavRoutes.Contacts.route) { Liked()  }
        }
        BottomNavigationBar(navController = navController)
    }
}
 
@Composable
fun BottomNavigationBar(navController: NavController) {
    NavigationBar {
        val backStackEntry by navController.currentBackStackEntryAsState()
        val currentRoute = backStackEntry?.destination?.route
 
        NavBarItems.BarItems.forEach { navItem ->
            NavigationBarItem(
                selected = currentRoute == navItem.route,
                onClick = {
                    navController.navigate(navItem.route) {
                        popUpTo(navController.graph.findStartDestination().id) {saveState = true}
                        launchSingleTop = true
                        restoreState = true
                    }
                },
                icon = {
                    Icon(imageVector = navItem.image,
                        contentDescription = navItem.title)
                },
                label = {
                    Text(text = navItem.title)
                }
            )
        }
    }
}
 
object NavBarItems {
    val BarItems = listOf(
        BarItem(
            title = "Home",
            image = Icons.Filled.Home,
            route = "home"
        ),
        BarItem(
            title = "Liked",
            image = Icons.Filled.Face,
            route = "liked"
        )
    )
}
 
data class BarItem(
    val title: String,
    val image: ImageVector,
    val route: String
)
 
@Composable
fun Home(vm: RandomDogViewModel = viewModel(), modifier: Modifier = Modifier) {
    Column(modifier = Modifier.fillMaxSize())
    {
        AsyncImage(model = vm.randomDog?.url, contentDescription = null, modifier = Modifier.size(200.dp, 200.dp))
        Spacer(modifier = Modifier.fillMaxHeight(0.9f))
        Row(modifier = Modifier.fillMaxWidth().background(Color.DarkGray).height(70.dp), horizontalArrangement = Arrangement.SpaceAround)
        {
            Button(onClick = { vm.fetchRandomDog() }) {
                val image = painterResource(R.drawable.refresh)
                Image(painter = image, contentDescription = null, modifier = Modifier.size(50.dp, 50.dp))
            }
            IconButton(onClick = { vm.addDog() }) {
                Icon(Icons.Filled.Favorite, contentDescription = "Информация о приложении", modifier = Modifier.size(80.dp),
                	tint = Color.Red)
            }
        }
    }
}

@Composable
fun Liked(vm: RandomDogViewModel = viewModel()){
    val dogs = vm.getDogs()
    LazyColumn(
                Modifier.fillMaxSize()
            ){
                item { Text("Собаки", fontSize = 29.sp) }
                items(dogs){dog -> AsyncImage(model = dog.url, contentDescription = null)}
            }
}
 
sealed class NavRoutes(val route: String) {
    object Home : NavRoutes("home")
    object Liked : NavRoutes("liked")
}
```





RandomDog
```
@Serializable
data class RandomDog (
    val fileSizeBytes: Int,
    val url: String
)

```


RandomDogRepository
```
interface RandomDogRepository {
    suspend fun getRandomDog(): RandomDog
}

class RandomDogRepositoryImpl (
    private val service: ApiServiceImpl = ApiServiceImpl(),
    private val dogDao: DogDao
) : RandomDogRepository {

    override suspend fun getRandomDog() : RandomDog {
        val response = service.fetchRandomDog()
        return response
    }

    private val coroutineScope = CoroutineScope(Dispatchers.Main)
 
    val dogList: LiveData<List<Dog>> = dogDao.getDogs()

    fun getDogs(): LiveData<List<Dog>>{
    	return dogDao.getDogs()
    }

    fun addDog(Dog: Dog) {
        coroutineScope.launch(Dispatchers.IO) {
            dogDao.addDog(Dog)
        }
    }
 
    fun deleteDog(id:Int) {
        coroutineScope.launch(Dispatchers.IO) {
            dogDao.deleteDog(id)
        }
    }
}  
```



RandomDogViewModel
```
class RandomDogViewModel(application: Application) : ViewModel() {
    private val repo: RandomDogRepositoryImpl
    var randomDog by mutableStateOf<RandomDog?>(null)
    val dogList: LiveData<List<Dog>>

    init {
        val dogDb = DogRoomDatabase.getInstance(application)
        val dogDao = dogDb.dogDao()
        repo = RandomDogRepositoryImpl()
        dogList = repo.dogList
    }

    fun fetchRandomDog() {
        viewModelScope.launch { randomDog = repo.getRandomDog() }
    }
 

    fun addDog() {
        repo.addDog(Dog(randomDog!!.url))
    }
    fun deleteDog(id: Int) {
        repo.deleteDog(id)
    }
    fun getDogs() {
	return repo.getDogs()
    }
}
```



DogRoomDatabase
```
@Database(entities = [(Dog::class)], version = 1)
abstract class DogRoomDatabase: RoomDatabase() {
 
    abstract fun dogDao(): DogDao
 
    companion object {
        private var INSTANCE: DogRoomDatabase? = null
        fun getInstance(context: Context): DogRoomDatabase {
 
            synchronized(this) {
                var instance = INSTANCE
                if (instance == null) {
                    instance = Room.databaseBuilder(
                        context.applicationContext,
                        UserRoomDatabase::class.java,
                        "dogdb"
                    ).fallbackToDestructiveMigration().build()
                    INSTANCE = instance
                }
                return instance
            }
        }
    }
}
```


DogDao
```
@Dao
interface DogDao {
    @Query("SELECT * FROM dogs")
    fun getDogs(): LiveData<List<Dog>>
 
    @Insert
    fun addDog(dog: Dog)
 
    @Query("DELETE FROM dogs WHERE id = :id")
    fun deleteDog(id:Int)
}
```

