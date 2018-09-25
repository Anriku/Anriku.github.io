---
layout:     post
title:      Android大量数据加载—Paging的使用
subtitle:   Android Paging的使用
date:       2018-09-25
author:     Anriku
header-img: img/2018_09_25_post.jpg
catalog: true
tags:
    - Android
    - Jetpack
    - Paging
---

Paging主要是用来结合RecyclerView进行使用的。**它的作用是能够逐渐地、优雅地加载所需要加载的数据。也就是一种分页方案。**



Paging每次只会加载总数据的一小部分。因此它有下面的两个优点：

* 数据加载要求更小的带宽以及更少的系统资源。
* 在资源发生改变的情况下，app依然能够很快的做出响应。



# Paging主要的类介绍

### PagedList

这个类是用来存储加载的数据。PagedList中所需要的数据都是从下面要讲的DataSource中进行加载的。



### DataSource

DataSource顾名思义就是数据来源。这类提供加载所需的数据。也就是在这个类中进行数据的获取操作。数据源可以是DataBase也可以是服务器。

DataSource的三个子类：

* PositionalDataSource: 主要用于加载数据可数有限的数据。比如加载本地数据库，这种情况下用户可以通过比如说像通讯录按姓的首字母查询的情况。能够跳转到任意的位置。
* ItemKeyedDataSource:主要用于加载逐渐增加的数据。比如说网络请求的数据随着不断的请求得到的数据越来越多。然后它适用的情况就是通过N-1item的数据来获取Nitem数据的情况。比如说Github的api。
* PageKeyedDataSource:这个和ItemKeyedDataSource有些相似，都是针对那种不断增加的数据。这里网络请求得到数据是分页的。比如说知乎日报的news的api。



### DataSource.Factory

这个接口的实现类主要是用来获取的DataSource的。



### PagedListAdapter

这个Adapter继承自RecyclerView.Adapter。如果要使用Paging，就需要让实现的RecyclerView的Adapter继承自PagedListAdapter。这个抽象类实现关于PagedList相关的东西。



### LivePagedListBuilder

通过这个类来生成对应的PagedList。



# 准备

```kotlin
// RxJava
implementation "io.reactivex.rxjava2:rxjava:2.2.2"
implementation "io.reactivex.rxjava2:rxandroid:2.1.0"

// Retrofit
implementation "com.squareup.retrofit2:adapter-rxjava2:2.4.0"
implementation "com.squareup.retrofit2:converter-gson:2.4.0"
implementation "com.squareup.retrofit2:retrofit:2.4.0"

// ViewModel and LiveData
def lifecycle_version = "1.1.1"
implementation "android.arch.lifecycle:extensions:$lifecycle_version"
kapt "android.arch.lifecycle:compiler:$lifecycle_version"

// Room
def room_version = "1.1.1"
implementation "android.arch.persistence.room:runtime:$room_version"
kapt "android.arch.persistence.room:compiler:$room_version"
implementation "android.arch.persistence.room:rxjava2:$room_version"

// Paging
def paging_version = "1.0.0"
implementation "android.arch.paging:runtime:$paging_version"

// Glide
implementation "com.github.bumptech.glide:glide:4.8.0"
kapt 'com.github.bumptech.glide:compiler:4.8.0'
```

应该大家都清楚这些库吧。这里就不一一解释了。**这里进行举例说明会用到Room、ViewModel、LiveData，请还不了解的朋友去看我的另外几篇博客。**



# PositionalDataSource的使用

首先是关于数据库的准备，内容很简单就不说了。

```kotlin
@Entity
data class Person(@PrimaryKey(autoGenerate = true) val id: Int, val name: String)
```



```kotlin
@Dao
interface PersonDao {
    @Insert
    fun insertPerson(person: Person)

    @Insert
    fun insertPersons(persons: List<Person>)

    @Delete
    fun deletePerson(person: Person)

    @Query("SELECT * FROM Person ORDER BY name COLLATE NOCASE ASC")
    fun getAllPersons(): DataSource.Factory<Int, Person>
}
```

关于Dao这里需要解释一下。getAllPersons返回一个**DataSource.Factory**。在之后会通过LivePagedListBuilder来构建PagedList。



```kotlin
@Database(entities = [Person::class], version = 1)
abstract class PersonDatabase : RoomDatabase() {
    abstract fun personDao(): PersonDao

    companion object {
        private var INSTANCE: PersonDatabase? = null

        fun get(context: Context): PersonDatabase {
            if (INSTANCE == null) {
                INSTANCE = Room.databaseBuilder(context, PersonDatabase::class.java, 	
                                                "PersonDatabase")
                        .addCallback(object : RoomDatabase.Callback() {
                            override fun onCreate(db: SupportSQLiteDatabase) {
                                fillDatabase(context)
                            }
                        })
                        .build()
            }
            return INSTANCE!!
        }

        private fun fillDatabase(context: Context) {
            ioThread {
                CHEESE_DATA.map {
                    get(context).personDao().insertPerson(Person(0, it))
                }
            }
        }
    }
}

private val EXECUTOR = Executors.newSingleThreadExecutor()

fun ioThread(f: () -> Unit) {
    EXECUTOR.execute(f)
}

private val CHEESE_DATA = arrayListOf(
        "Abbaye de Belloc", "Abbaye du Mont des Cats", "Abertam", "Abondance", "Ackawi",
        "Acorn", "Adelost", "Affidelice au Chablis", "Afuega'l Pitu", "Airag", "Airedale",
        "Aisy Cendre", "Allgauer Emmentaler", "Alverca", "Ambert", "American Cheese",
        "Ami du Chambertin", "Anejo Enchilado", "Anneau du Vic-Bilh", "Anthoriro", "Appenzell",
        "Aragon", "Ardi Gasna", "Ardrahan", "Armenian String", "Aromes au Gene de Marc",
        "Asadero", "Asiago", "Aubisque Pyrenees", "Autun", "Avaxtskyr", "Baby Swiss",
        "Babybel", "Baguette Laonnaise", "Bakers", "Baladi", "Balaton", "Bandal", "Banon",
        "Barry's Bay Cheddar", "Basing", "Basket Cheese", "Bath Cheese", "Bavarian Bergkase",
        "Baylough", "Beaufort", "Beauvoorde", "Beenleigh Blue", "Beer Cheese", "Bel Paese",
        "Bergader", "Bergere Bleue", "Berkswell", "Beyaz Peynir", "Bierkase", "Bishop Kennedy",
        "Blarney", "Bleu d'Auvergne", "Bleu de Gex", "Bleu de Laqueuille",
        "Bleu de Septmoncel", "Bleu Des Causses", "Blue", "Blue Castello", "Blue Rathgore",
        "Blue Vein (Australian)", "Blue Vein Cheeses", "Bocconcini", "Bocconcini (Australian)",
        "Boeren Leidenkaas", "Bonchester", "Bosworth", "Bougon", "Boule Du Roves",
        "Boulette d'Avesnes", "Boursault", "Boursin", "Bouyssou", "Bra", "Braudostur",
        "Breakfast Cheese", "Brebis du Lavort", "Brebis du Lochois", "Brebis du Puyfaucon",
        "Bresse Bleu", "Brick", "Brie", "Brie de Meaux", "Brie de Melun", "Brillat-Savarin",
        "Brin", "Brin d' Amour", "Brin d'Amour", "Brinza (Burduf Brinza)",
        "Briquette de Brebis", "Briquette du Forez", "Broccio", "Broccio Demi-Affine",
        "Brousse du Rove", "Bruder Basil", "Brusselae Kaas (Fromage de Bruxelles)", "Bryndza",
        "Buchette d'Anjou", "Buffalo", "Burgos", "Butte", "Butterkase", "Button (Innes)",
        "Buxton Blue", "Cabecou", "Caboc", "Cabrales", "Cachaille", "Caciocavallo", "Caciotta",
        "Caerphilly", "Cairnsmore", "Calenzana", "Cambazola", "Camembert de Normandie",
        "Canadian Cheddar", "Canestrato", "Cantal", "Caprice des Dieux", "Capricorn Goat",
        "Capriole Banon", "Carre de l'Est", "Casciotta di Urbino", "Cashel Blue", "Castellano",
        "Castelleno", "Castelmagno", "Castelo Branco", "Castigliano", "Cathelain",
        "Celtic Promise", "Cendre d'Olivet", "Cerney", "Chabichou", "Chabichou du Poitou",
        "Chabis de Gatine", "Chaource", "Charolais", "Chaumes", "Cheddar",
        "Cheddar Clothbound", "Cheshire", "Chevres", "Chevrotin des Aravis", "Chontaleno",
        "Civray", "Coeur de Camembert au Calvados", "Coeur de Chevre", "Colby", "Cold Pack",
        "Comte", "Coolea", "Cooleney", "Coquetdale", "Corleggy", "Cornish Pepper",
        "Cotherstone", "Cotija", "Cottage Cheese", "Cottage Cheese (Australian)",
        "Cougar Gold", "Coulommiers", "Coverdale", "Crayeux de Roncq", "Cream Cheese",
        "Cream Havarti", "Crema Agria", "Crema Mexicana", "Creme Fraiche", "Crescenza",
        "Croghan", "Crottin de Chavignol", "Crottin du Chavignol", "Crowdie", "Crowley",
        "Cuajada", "Curd", "Cure Nantais", "Curworthy", "Cwmtawe Pecorino",
        "Cypress Grove Chevre", "Danablu (Danish Blue)", "Danbo", "Danish Fontina",
        "Daralagjazsky", "Dauphin", "Delice des Fiouves", "Denhany Dorset Drum", "Derby",
        "Dessertnyj Belyj", "Devon Blue", "Devon Garland", "Dolcelatte", "Doolin",
        "Doppelrhamstufel", "Dorset Blue Vinney", "Double Gloucester", "Double Worcester",
        "Dreux a la Feuille", "Dry Jack", "Duddleswell", "Dunbarra", "Dunlop", "Dunsyre Blue",
        "Duroblando", "Durrus", "Dutch Mimolette (Commissiekaas)", "Edam", "Edelpilz",
        "Emental Grand Cru", "Emlett", "Emmental", "Epoisses de Bourgogne", "Esbareich",
        "Esrom", "Etorki", "Evansdale Farmhouse Brie", "Evora De L'Alentejo", "Exmoor Blue",
        "Explorateur", "Feta", "Feta (Australian)", "Figue", "Filetta", "Fin-de-Siecle",
        "Finlandia Swiss", "Finn", "Fiore Sardo", "Fleur du Maquis", "Flor de Guia",
        "Flower Marie", "Folded", "Folded cheese with mint", "Fondant de Brebis",
        "Fontainebleau", "Fontal", "Fontina Val d'Aosta", "Formaggio di capra", "Fougerus",
        "Four Herb Gouda", "Fourme d' Ambert", "Fourme de Haute Loire", "Fourme de Montbrison",
        "Fresh Jack", "Fresh Mozzarella", "Fresh Ricotta", "Fresh Truffles", "Fribourgeois",
        "Friesekaas", "Friesian", "Friesla", "Frinault", "Fromage a Raclette", "Fromage Corse",
        "Fromage de Montagne de Savoie", "Fromage Frais", "Fruit Cream Cheese",
        "Frying Cheese", "Fynbo", "Gabriel", "Galette du Paludier", "Galette Lyonnaise",
        "Galloway Goat's Milk Gems", "Gammelost", "Gaperon a l'Ail", "Garrotxa", "Gastanberra",
        "Geitost", "Gippsland Blue", "Gjetost", "Gloucester", "Golden Cross", "Gorgonzola",
        "Gornyaltajski", "Gospel Green", "Gouda", "Goutu", "Gowrie", "Grabetto", "Graddost",
        "Grafton Village Cheddar", "Grana", "Grana Padano", "Grand Vatel",
        "Grataron d' Areches", "Gratte-Paille", "Graviera", "Greuilh", "Greve",
        "Gris de Lille", "Gruyere", "Gubbeen", "Guerbigny", "Halloumi",
        "Halloumy (Australian)", "Haloumi-Style Cheese", "Harbourne Blue", "Havarti",
        "Heidi Gruyere", "Hereford Hop", "Herrgardsost", "Herriot Farmhouse", "Herve",
        "Hipi Iti", "Hubbardston Blue Cow", "Hushallsost", "Iberico", "Idaho Goatster",
        "Idiazabal", "Il Boschetto al Tartufo", "Ile d'Yeu", "Isle of Mull", "Jarlsberg",
        "Jermi Tortes", "Jibneh Arabieh", "Jindi Brie", "Jubilee Blue", "Juustoleipa",
        "Kadchgall", "Kaseri", "Kashta", "Kefalotyri", "Kenafa", "Kernhem", "Kervella 	 Affine",
        "Kikorangi", "King Island Cape Wickham Brie", "King River Gold", "Klosterkaese",
        "Knockalara", "Kugelkase", "L'Aveyronnais", "L'Ecir de l'Aubrac", "La Taupiniere",
        "La Vache Qui Rit", "Laguiole", "Lairobell", "Lajta", "Lanark Blue", "Lancashire",
        "Langres", "Lappi", "Laruns", "Lavistown", "Le Brin", "Le Fium Orbo", "Le Lacandou",
        "Le Roule", "Leafield", "Lebbene", "Leerdammer", "Leicester", "Leyden", "Limburger",
        "Lincolnshire Poacher", "Lingot Saint Bousquet d'Orb", "Liptauer", "Little Rydings",
        "Livarot", "Llanboidy", "Llanglofan Farmhouse", "Loch Arthur Farmhouse",
        "Loddiswell Avondale", "Longhorn", "Lou Palou", "Lou Pevre", "Lyonnais", "Maasdam",
        "Macconais", "Mahoe Aged Gouda", "Mahon", "Malvern", "Mamirolle", "Manchego",
        "Manouri", "Manur", "Marble Cheddar", "Marbled Cheeses", "Maredsous", "Margotin",
        "Maribo", "Maroilles", "Mascares", "Mascarpone", "Mascarpone (Australian)",
        "Mascarpone Torta", "Matocq", "Maytag Blue", "Meira", "Menallack Farmhouse",
        "Menonita", "Meredith Blue", "Mesost", "Metton (Cancoillotte)", "Meyer Vintage Gouda",
        "Mihalic Peynir", "Milleens", "Mimolette", "Mine-Gabhar", "Mini Baby Bells", "Mixte",
        "Molbo", "Monastery Cheeses", "Mondseer", "Mont D'or Lyonnais", "Montasio",
        "Monterey Jack", "Monterey Jack Dry", "Morbier", "Morbier Cru de Montagne",
        "Mothais a la Feuille", "Mozzarella", "Mozzarella (Australian)",
        "Mozzarella di Bufala", "Mozzarella Fresh, in water", "Mozzarella Rolls", "Munster",
        "Murol", "Mycella", "Myzithra", "Naboulsi", "Nantais", "Neufchatel",
        "Neufchatel (Australian)", "Niolo", "Nokkelost", "Northumberland", "Oaxaca",
        "Olde York", "Olivet au Foin", "Olivet Bleu", "Olivet Cendre",
        "Orkney Extra Mature Cheddar", "Orla", "Oschtjepka", "Ossau Fermier", "Ossau-Iraty",
        "Oszczypek", "Oxford Blue", "P'tit Berrichon", "Palet de Babligny", "Paneer", "Panela",
        "Pannerone", "Pant ys Gawn", "Parmesan (Parmigiano)", "Parmigiano Reggiano",
        "Pas de l'Escalette", "Passendale", "Pasteurized Processed", "Pate de Fromage",
        "Patefine Fort", "Pave d'Affinois", "Pave d'Auge", "Pave de Chirac", "Pave du Berry",
        "Pecorino", "Pecorino in Walnut Leaves", "Pecorino Romano", "Peekskill Pyramid",
        "Pelardon des Cevennes", "Pelardon des Corbieres", "Penamellera", "Penbryn",
        "Pencarreg", "Perail de Brebis", "Petit Morin", "Petit Pardou", "Petit-Suisse",
        "Picodon de Chevre", "Picos de Europa", "Piora", "Pithtviers au Foin",
        "Plateau de Herve", "Plymouth Cheese", "Podhalanski", "Poivre d'Ane", "Polkolbin",
        "Pont l'Eveque", "Port Nicholson", "Port-Salut", "Postel", "Pouligny-Saint-Pierre",
        "Pourly", "Prastost", "Pressato", "Prince-Jean", "Processed Cheddar", "Provolone",
        "Provolone (Australian)", "Pyengana Cheddar", "Pyramide", "Quark",
        "Quark (Australian)", "Quartirolo Lombardo", "Quatre-Vents", "Quercy Petit",
        "Queso Blanco", "Queso Blanco con Frutas --Pina y Mango", "Queso de Murcia",
        "Queso del Montsec", "Queso del Tietar", "Queso Fresco", "Queso Fresco (Adobera)",
        "Queso Iberico", "Queso Jalapeno", "Queso Majorero", "Queso Media Luna",
        "Queso Para Frier", "Queso Quesadilla", "Rabacal", "Raclette", "Ragusano", "Raschera",
        "Reblochon", "Red Leicester", "Regal de la Dombes", "Reggianito", "Remedou",
        "Requeson", "Richelieu", "Ricotta", "Ricotta (Australian)", "Ricotta Salata", "Ridder",
        "Rigotte", "Rocamadour", "Rollot", "Romano", "Romans Part Dieu", "Roncal", "Roquefort",
        "Roule", "Rouleau De Beaulieu", "Royalp Tilsit", "Rubens", "Rustinu", "Saaland Pfarr",
        "Saanenkaese", "Saga", "Sage Derby", "Sainte Maure", "Saint-Marcellin",
        "Saint-Nectaire", "Saint-Paulin", "Salers", "Samso", "San Simon", "Sancerre",
        "Sap Sago", "Sardo", "Sardo Egyptian", "Sbrinz", "Scamorza", "Schabzieger", "Schloss",
        "Selles sur Cher", "Selva", "Serat", "Seriously Strong Cheddar", "Serra da Estrela",
        "Sharpam", "Shelburne Cheddar", "Shropshire Blue", "Siraz", "Sirene", "Smoked Gouda",
        "Somerset Brie", "Sonoma Jack", "Sottocenare al Tartufo", "Soumaintrain",
        "Sourire Lozerien", "Spenwood", "Sraffordshire Organic", "St. Agur Blue Cheese",
        "Stilton", "Stinking Bishop", "String", "Sussex Slipcote", "Sveciaost", "Swaledale",
        "Sweet Style Swiss", "Swiss", "Syrian (Armenian String)", "Tala", "Taleggio", "Tamie",
        "Tasmania Highland Chevre Log", "Taupiniere", "Teifi", "Telemea", "Testouri",
        "Tete de Moine", "Tetilla", "Texas Goat Cheese", "Tibet", "Tillamook Cheddar",
        "Tilsit", "Timboon Brie", "Toma", "Tomme Brulee", "Tomme d'Abondance",
        "Tomme de Chevre", "Tomme de Romans", "Tomme de Savoie", "Tomme des Chouans", "Tommes",
        "Torta del Casar", "Toscanello", "Touree de L'Aubier", "Tourmalet",
        "Trappe (Veritable)", "Trois Cornes De Vendee", "Tronchon", "Trou du Cru", "Truffe",
        "Tupi", "Turunmaa", "Tymsboro", "Tyn Grug", "Tyning", "Ubriaco", "Ulloa",
        "Vacherin-Fribourgeois", "Valencay", "Vasterbottenost", "Venaco", "Vendomois",
        "Vieux Corse", "Vignotte", "Vulscombe", "Waimata Farmhouse Blue",
        "Washed Rind Cheese (Australian)", "Waterloo", "Weichkaese", "Wellington",
        "Wensleydale", "White Stilton", "Whitestone Farmhouse", "Wigmore", "Woodside Cabecou",
        "Xanadu", "Xynotyro", "Yarg Cornish", "Yarra Valley Pyramid", "Yorkshire Blue",
        "Zamorano", "Zanetti Grana Padano", "Zanetti Parmigiano Reggiano")

```

名字很长，为了方便大家直接拿去用这里全部放上去了。**这里实现的是在数据库初次构建的时候将下面的数据存储在数据库中**



下面我们构建一个ViewModel：

```kotlin
class PersonViewModel(application: Application) : AndroidViewModel(application) {
    companion object {
        private const val PAGE_SIZE = 30
        private const val ENABLE_PLACEHOLDER = true
    }
    
    private val mPersonDao = PersonDatabase.get(application).personDao()

    val persons = LivePagedListBuilder(mPersonDao.getAllPersons(), PagedList
            .Config.Builder()
            .setPageSize(PAGE_SIZE)
            .setEnablePlaceholders(ENABLE_PLACEHOLDER).build()).build()
}
```

**这里先通过RoomDatabase来获取一个PersonDao对象。再通过LivePagedListBuilder获取一个PagedList。**

**PagedList.Config**是用于对PagedList进行构建配置的类。其中**PAGE_SIZE**用于指定每页数据量。**ENABLE_PLACEHOLDER**表示是否将未加载的数据以null存储在在PageList中。具体的效果试了就知道了。



然后，构建一个PagedListAdapter：

```kotlin
class PersonRecAdapter(val context: Context) : PagedListAdapter<Person, PersonRecAdapter.PersonViewHolder>(diffCallBack) {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PersonViewHolder {
        return 
        PersonViewHolder(LayoutInflater.from(context).inflate(R.layout.person_rec_item, 
                                                              parent, false))
    }

    override fun onBindViewHolder(holder: PersonViewHolder, position: Int) {
        holder.bindTo(getItem(position))
    }

    companion object {
        private val diffCallBack = object : DiffUtil.ItemCallback<Person>() {

            override fun areItemsTheSame(oldItem: Person, newItem: Person): Boolean {
                return oldItem.id == newItem.id
            }

            override fun areContentsTheSame(oldItem: Person, newItem: Person): Boolean {
                return oldItem == newItem
            }

            override fun getChangePayload(oldItem: Person, newItem: Person): Any? {
                return null
            }
        }
    }

    class PersonViewHolder(itemView: View): RecyclerView.ViewHolder(itemView) {
        val id = itemView.findViewById<TextView>(R.id.id)
        val name = itemView.findViewById<TextView>(R.id.name)

        fun bindTo(person: Person?){
            Log.d("PersonViewHolder", person.toString())
            id.text = person!!.id.toString()
            name.text = person.name
        }
    }
}
```

在对下面的内容解释之前。先说一下onBindViewHolder有两个重载方法。一个是我们熟悉的**onBindViewHolder(holder: PersonViewHolder, position: Int)**，另一个是**onBindViewHolder(holder: PersonViewHolder, position: Int, payloads: MutableList&lt;Any>)**。其中带payloads的方法默认实现是调用不带payloads的。



PagedListAdapter需要说的地方就是它接收一个**DiffUtil.ItemCallback**参数进行对象的构建。这个用于在PagedListAdapter的**PageList发生变化后**进行比对。**如果areItemsTheSame返回true、areContentsTheSame返回false就会先调用带payloads的onBindViewHolder再根据前面的onBindViewHolder的情况看是否调用不带payloads的onBindViewHolder。这里怎么判断是否改变就是根据具体情况实现具体的逻辑了。**



OK，最后来看看在Activity中的实现吧：

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var mPersonViewModel: PersonViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        mPersonViewModel = ViewModelProviders.of(this).get(PersonViewModel::class.java)
        val adapter = PersonRecAdapter(this)
        rv.adapter = adapter
        mPersonViewModel.persons.observe(this, Observer(adapter::submitList))
    }
}
```

很简单。给ViewModel中的persons这个LiveData&lt;PagedList&lt;Person>>进行添加一个观察者。**在persons数据发生变化后，就调用PersonRecAdapter的submitList方法进行数据的添加。**



Activity的xml如下**(下面的讲解中的Actvity是一样的就不会再列出了)**：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".paging.positional.MainActivity">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/rv"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:scrollbars="vertical"
        android:text="Hello World!"
        app:layoutManager="android.support.v7.widget.LinearLayoutManager"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</android.support.constraint.ConstraintLayout>
```



**这里结合Room使用就完了。然后大家肯定会惊讶从头到尾都没有看到PositionalDataSource的影子。其实这是Room替我们生成了一个继承了PositionalDataSource的类——LimitOffsetDataSource的。这里就不列出来了。有兴趣的可以自己去看一下这个类。**





# ItemKeyedDataSource

**这个DataSource用在N-1的item的内容中的某个信息指向N的item。相当于Item之间通过链表链着一样。**Github获取帐号的API就是典型的这样的API。因此这里以Github的API进行举例。

**https://api.github.com/users?since=0?per_page=30**这是github获取帐号的api。**其中需要注意一点的就是如果一个IP地址对这个api使用超过一定的流量，会有段时间静止访问**



下面开是写代码了。

GithubService:

```kotlin
interface GitHubService {
    @GET("users")
    fun getGithubAccount(@Query("since") id: Long, @Query("per_page") perPage: Int):
            Observable<List<GithubAccount>>
}
```

ApiGenerate:

```kotlin
object ApiGenerate {
    private val retrofit = Retrofit.Builder()
            .baseUrl("https://api.github.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
            .build()
            
    fun getGitHubService(): GitHubService = retrofit.create(GitHubService::class.java)
}
```

GithubAccount:

```kotlin
data class GithubAccount(
        var login: String? = null,
        var id: Int = 0,
        var node_id: String? = null,
        var avatar_url: String? = null,
        var gravatar_id: String? = null,
        var url: String? = null,
        var html_url: String? = null,
        var followers_url: String? = null,
        var following_url: String? = null,
        var gists_url: String? = null,
        var starred_url: String? = null,
        var subscriptions_url: String? = null,
        var organizations_url: String? = null,
        var repos_url: String? = null,
        var events_url: String? = null,
        var received_events_url: String? = null,
        var type: String? = null,
        var isSite_admin: Boolean = false)
```

上面相信都没有问题。

ExecuteOnceObserver:

```kotlin
class ExecuteOnceObserver<T>(val onExecuteOnceNext: (T) -> Unit = {},
                             val onExecuteOnceComplete: () -> Unit = {},
                             val onExecuteOnceError: (Throwable) -> Unit = {}) : Observer<T> {
	private var mDisposable: Disposable? = null

    override fun onComplete() {
        onExecuteOnceComplete()
    }

    override fun onSubscribe(d: Disposable) {
        mDisposable = d
    }

    override fun onNext(t: T) {
        try {
            onExecuteOnceNext(t)
            this.onComplete()
        } catch (e: Throwable) {
            this.onError(e)
        } finally {
            if (mDisposable != null && !mDisposable!!.isDisposed) {
                mDisposable!!.dispose()
            }
        }
    }
    override fun onError(e: Throwable) {
        onExecuteOnceError(e)
    }
}
```

**这是一个用于执行一次onNext就被销毁的Observer工具类。**



**下面是重点**

ByItemDataSource：

```kotlin
class ByItemDataSource : ItemKeyedDataSource<Long, GithubAccount>() {

    private val mGitHubService by lazy {
        ApiGenerate.getGitHubService()
    }

    override fun loadInitial(params: LoadInitialParams<Long>, callback: 
    LoadInitialCallback<GithubAccount>) {
        mGitHubService.getGithubAccount(0, params.requestedLoadSize)
                .observeOn(AndroidSchedulers.mainThread())
                .subscribeOn(Schedulers.newThread())
                .subscribe(ExecuteOnceObserver({
                    callback.onResult(it)
                }))
    }

    override fun loadAfter(params: LoadParams<Long>, callback: 
    LoadCallback<GithubAccount>) {
        mGitHubService.getGithubAccount(params.key, params.requestedLoadSize)
                .observeOn(AndroidSchedulers.mainThread())
                .subscribeOn(Schedulers.newThread())
                .subscribe(ExecuteOnceObserver(onExecuteOnceNext = {
                    callback.onResult(it)
                }))
    }

    override fun loadBefore(params: LoadParams<Long>, callback: 
    LoadCallback<GithubAccount>) {
		//由于这里不需要向上加载因此省略此处
    }

    override fun getKey(item: GithubAccount): Long = item.id.toLong()
}
```

ItemKeyedDataSource的子类需要实现loadInitial、loadAfter、loadBefore和getKey方法。它们分别的作用如下：

* loadInitial：此方法之后在用DataSource构建PageList的时候才会调用一次。用于进行加载初始化。
* loadAfter：在每次RecyclerView滑动到`底部`没有数据的时候就会调用此方法进行数据的加载。
* loadBefore：在每次RecyclerView滑动到`顶部`没有数据的时候就会调用此方法进行数据的加载。
* getKey: 这返回下一个loadAfter调用所需要用到的key。**就相当于链表的指针。**

**其中三个load方法都是通过LoadInitialCallback、LoadCallback来将数据传给PagedList的。**



经过上面每个方法的解释应该没有问题。**Github的api就相当于id作为了链表的指针了。**



ByItemDataSourceFactory:

```kotlin
class ByItemDataSourceFactory : DataSource.Factory<Long, GithubAccount>() {
    override fun create(): DataSource<Long, GithubAccount> = ByItemDataSource()
}
```

就是将上面DataSource作一个返回很简单。



ByItemAdapter:

```kotlin
class ByItemAdapter : PagedListAdapter<GithubAccount, ByItemAdapter.ByItemViewHolder>(diffCallback) {

    companion object {
        val diffCallback = object : DiffUtil.ItemCallback<GithubAccount>() {
            override fun areItemsTheSame(oldItem: GithubAccount, newItem: GithubAccount): Boolean {
                return oldItem.id == newItem.id
            }

            override fun areContentsTheSame(oldItem: GithubAccount, newItem: GithubAccount): Boolean {
                return oldItem == newItem
            }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ByItemViewHolder {
        return ByItemViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.by_item_rec_adapter, parent, false))
    }

    override fun onBindViewHolder(holder: ByItemViewHolder, position: Int) {
        holder.bindTo(getItem(position))
    }

    class ByItemViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {

        private val mId: TextView = itemView.findViewById(R.id.id)
        private val mName: TextView = itemView.findViewById(R.id.name)

        fun bindTo(account: GithubAccount?) {
            account?.let {
                mId.text = it.id.toString()
                mName.text = it.login
            }
        }
    }
}
```

这个和前面讲PositionalDataSource处的Adapter重点是一样的这里就不重复了。



下面是在Activity中的使用

```kotlin
class ByItemActivity : AppCompatActivity() {

    private lateinit var mByItemViewModel: ByItemViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_by_item)
        mByItemViewModel = ViewModelProviders.of(this).get(ByItemViewModel::class.java)

        val adapter = ByItemAdapter()
        rv.adapter = adapter
        mByItemViewModel.accounts.observe(this, Observer(adapter::submitList))
    }
}
```

很简单和前面一样的设置监听器和数据。



# PageKeyedDataSource的使用

**这个用服务器本身是分页实现的，然后我们通过其返回的数据每一页的数据得到下一页的key。**典型的知乎日报的api就是这样的。

这是知乎日报查看过往消息的api：

**https://news-at.zhihu.com/api/4/news/before/20180823**



NewsService:

```kotlin
interface NewsService {
    @GET("before/{time}")
    fun getNews(@Path("time")time: Long): Observable<News>
}
```

News:

```kotlin
class News(var date: String = "",
           var stories: List<StoriesBean> = emptyList()) {

    class StoriesBean {
        var type: Int = 0
        var id: Int = 0
        var ga_prefix: String? = null
        var title: String? = null
        var images: List<String>? = null
    }
}
```

ByPageDataSource:

```kotlin
class ByPageDataSource : PageKeyedDataSource<Long, News.StoriesBean>() {

    private lateinit var mNewsService: NewsService
    private val mDate = Calendar.getInstance().apply {
        add(Calendar.DATE, 1)
    }
    
    override fun loadInitial(params: LoadInitialParams<Long>, callback: 
                             LoadInitialCallback<Long, News.StoriesBean>) {
        mNewsService = ApiGenerate.getNewsService()
        mNewsService.getNews(SimpleDateFormat("yyyyMMdd", 
                                              Locale.CHINA).format(mDate.time).toLong())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribeOn(Schedulers.newThread())
                .subscribe(ExecuteOnceObserver(onExecuteOnceNext = {
                    callback.onResult(it.stories, 0, it.date.toLong())
                }))
    }

    override fun loadAfter(params: LoadParams<Long>, callback: LoadCallback<Long, 
                           News.StoriesBean>) {
        mNewsService.getNews(params.key)
                .observeOn(AndroidSchedulers.mainThread())
                .subscribeOn(Schedulers.newThread())
                .subscribe(ExecuteOnceObserver(onExecuteOnceNext = {
                    callback.onResult(it.stories, it.date.toLong())
                }))
    }

    override fun loadBefore(params: LoadParams<Long>, callback: LoadCallback<Long, 
                            News.StoriesBean>) {
		//这里不需要向上加载，因此无须实现
    }
}
```

对于PageKeyedDataSource的子类有三个要实现方法loadInitial、loadAfter和loadBefore。**其中三个方法的作用和ItemKeyedDataSource是一样的。只不过这里LoadInitialCallback、LoadCallback和ItemKeyedDataSource不一样。这个就请自己去它的不同api了。**



ByPageDataSourceFactory:

```kotlin
class ByPageDataSourceFactory : DataSource.Factory<Long, News.StoriesBean>() {
    override fun create(): DataSource<Long, News.StoriesBean> = ByPageDataSource()
}
```



ByPageViewModel:

```kotlin
class ByPageViewModel : ViewModel() {
    val stories = LivePagedListBuilder(ByPageDataSourceFactory(),
            PagedList.Config.Builder()
                    .setPageSize(30)
                    .setEnablePlaceholders(false).build()).build()
}
```

**这里我们设置的pageSize并没有用再DataSource中我们并没有使用到。但是这个值必须是个正数。**



Activity中的使用

```kotlin
class ByPageActivity : AppCompatActivity() {

    private lateinit var mByPageViewModel: ByPageViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_by_page)

        mByPageViewModel = ViewModelProviders.of(this).get(ByPageViewModel::class.java)
        val adapter = ByPageAdapter()
        rv.adapter = adapter
        mByPageViewModel.stories.observe(this, Observer(adapter::submitList))
    }
}
```



# 总结

到此为止围绕着三个DataSource都已将讲完了。经过大家的代码实践，这里在给大家贴上官方的一张图。相信大家不会对其中的关系不会太难理解：

![Paging 原理](http://oyil5gdc8.bkt.clouddn.com/511a702ae4af43cd.png)



# 参考

[官方文档](https://developer.android.com/topic/libraries/architecture/paging/)



*转载请注明链接*



