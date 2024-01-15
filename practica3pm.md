# Practica3_Persistenciadedatos
Instalar la dependencia de room en build.gradle.kts ubicado en el apartado de Gradle Scripts, en el apartado de dependencies:
buildscript { extra.apply { set("room_version", "2.5.2") } }

plugins { id("com.android.application") version "8.0.2" apply false id("com.android.library") version "8.0.2" apply false id("org.jetbrains.kotlin.android") version "1.8.21" apply false }

tasks.register("clean", Delete::class) { delete(rootProject.buildDir) }

Necesitamos ver los permisos en AndroidManifest
<application
    android:name=".InventoryApplication"
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true"
    android:theme="@style/Theme.Inventory"
    tools:targetApi="33">
    <activity
        android:name=".MainActivity"
        android:exported="true"
        android:label="@string/app_name"
        android:theme="@style/Theme.Inventory">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>
</application>
Contenido de la app
package com.example.inventory.data

import android.content.Context

/**

App container for Dependency injection. */ interface AppContainer { val itemsRepository: ItemsRepository }
/**

[AppContainer] implementation that provides instance of [OfflineItemsRepository] / class AppDataContainer(private val context: Context) : AppContainer { /*
Implementation for [ItemsRepository] */ override val itemsRepository: ItemsRepository by lazy { OfflineItemsRepository() } }
ITEM

package com.example.inventory.data

/**

Entity data class represents a single row in the database. */ class Item( val id: Int = 0, val name: String, val price: Double, val quantity: Int )
ITEMREPOSYTORIO

package com.example.inventory.data

/**

Repository that provides insert, update, delete, and retrieve of [Item] from a given data source. */ interface ItemsRepository
OFFLINE REPOSITOREY

package com.example.inventory.data

class OfflineItemsRepository : ItemsRepository

Cuerpo de la app en previw
package com.example.inventory.ui.home

import android.annotation.SuppressLint import androidx.compose.foundation.clickable import androidx.compose.foundation.layout.Arrangement import androidx.compose.foundation.layout.Column import androidx.compose.foundation.layout.Row import androidx.compose.foundation.layout.Spacer import androidx.compose.foundation.layout.fillMaxSize import androidx.compose.foundation.layout.fillMaxWidth import androidx.compose.foundation.layout.padding import androidx.compose.foundation.lazy.LazyColumn import androidx.compose.foundation.lazy.items import androidx.compose.material.icons.Icons import androidx.compose.material.icons.filled.Add import androidx.compose.material3.Card import androidx.compose.material3.CardDefaults import androidx.compose.material3.ExperimentalMaterial3Api import androidx.compose.material3.FloatingActionButton import androidx.compose.material3.Icon import androidx.compose.material3.MaterialTheme import androidx.compose.material3.Scaffold import androidx.compose.material3.Text import androidx.compose.material3.TopAppBarDefaults import androidx.compose.runtime.Composable import androidx.compose.ui.Alignment import androidx.compose.ui.Modifier import androidx.compose.ui.input.nestedscroll.nestedScroll import androidx.compose.ui.res.dimensionResource import androidx.compose.ui.res.stringResource import androidx.compose.ui.text.style.TextAlign import androidx.compose.ui.tooling.preview.Preview import androidx.compose.ui.unit.dp import com.example.inventory.InventoryTopAppBar import com.example.inventory.R import com.example.inventory.data.Item import com.example.inventory.ui.item.formatedPrice import com.example.inventory.ui.navigation.NavigationDestination import com.example.inventory.ui.theme.InventoryTheme

object HomeDestination : NavigationDestination { override val route = "home" override val titleRes = R.string.app_name }

/**

Entry route for Home screen */ @OptIn(ExperimentalMaterial3Api::class) @SuppressLint("UnusedMaterial3ScaffoldPaddingParameter") @Composable fun HomeScreen( navigateToItemEntry: () -> Unit, navigateToItemUpdate: (Int) -> Unit, modifier: Modifier = Modifier ) { val scrollBehavior = TopAppBarDefaults.enterAlwaysScrollBehavior()

Scaffold( modifier = modifier.nestedScroll(scrollBehavior.nestedScrollConnection), topBar = { InventoryTopAppBar( title = stringResource(HomeDestination.titleRes), canNavigateBack = false, scrollBehavior = scrollBehavior ) }, floatingActionButton = { FloatingActionButton( onClick = navigateToItemEntry, shape = MaterialTheme.shapes.medium, modifier = Modifier.padding(dimensionResource(id = R.dimen.padding_large)) ) { Icon( imageVector = Icons.Default.Add, contentDescription = stringResource(R.string.item_entry_title) ) } }, ) { innerPadding -> HomeBody( itemList = listOf(), onItemClick = navigateToItemUpdate, modifier = Modifier .padding(innerPadding) .fillMaxSize() ) } }

@Composable private fun HomeBody( itemList: List, onItemClick: (Int) -> Unit, modifier: Modifier = Modifier ) { Column( horizontalAlignment = Alignment.CenterHorizontally, modifier = modifier ) { if (itemList.isEmpty()) { Text( text = stringResource(R.string.no_item_description), textAlign = TextAlign.Center, style = MaterialTheme.typography.titleLarge ) } else { InventoryList( itemList = itemList, onItemClick = { onItemClick(it.id) }, modifier = Modifier.padding(horizontal = dimensionResource(id = R.dimen.padding_small)) ) } } }

@Composable private fun InventoryList( itemList: List, onItemClick: (Item) -> Unit, modifier: Modifier = Modifier ) { LazyColumn(modifier = modifier) { items(items = itemList, key = { it.id }) { item -> InventoryItem(item = item, modifier = Modifier .padding(dimensionResource(id = R.dimen.padding_small)) .clickable { onItemClick(item) }) } } }

@Composable private fun InventoryItem( item: Item, modifier: Modifier = Modifier ) { Card( modifier = modifier, elevation = CardDefaults.cardElevation(defaultElevation = 2.dp) ) { Column( modifier = Modifier.padding(dimensionResource(id = R.dimen.padding_large)), verticalArrangement = Arrangement.spacedBy(dimensionResource(id = R.dimen.padding_small)) ) { Row( modifier = Modifier.fillMaxWidth() ) { Text( text = item.name, style = MaterialTheme.typography.titleLarge, ) Spacer(Modifier.weight(1f)) Text( text = item.formatedPrice(), style = MaterialTheme.typography.titleMedium ) } Text( text = stringResource(R.string.in_stock, item.quantity), style = MaterialTheme.typography.titleMedium ) } } }

@Preview(showBackground = true) @Composable fun HomeBodyPreview() { InventoryTheme { HomeBody(listOf( Item(1, "Game", 100.0, 20), Item(2, "Pen", 200.0, 30), Item(3, "TV", 300.0, 50) ), onItemClick = {}) } }

@Preview(showBackground = true) @Composable fun HomeBodyEmptyListPreview() { InventoryTheme { HomeBody(listOf(), onItemClick = {}) } }

@Preview(showBackground = true) @Composable fun InventoryItemPreview() { InventoryTheme { InventoryItem( Item(1, "Game", 100.0, 20), ) } } HOME VIEW MODEL package com.example.inventory.ui.home

import androidx.lifecycle.ViewModel import com.example.inventory.data.Item

/**

ViewModel to retrieve all items in the Room database. */ class HomeViewModel : ViewModel() { companion object { private const val TIMEOUT_MILLIS = 5_000L } }
/**

Ui State for HomeScreen */ data class HomeUiState(val itemList: List = listOf())
Items detalles
package com.example.inventory.ui.item

import androidx.annotation.StringRes import androidx.compose.foundation.layout.Arrangement import androidx.compose.foundation.layout.Column import androidx.compose.foundation.layout.Row import androidx.compose.foundation.layout.Spacer import androidx.compose.foundation.layout.fillMaxWidth import androidx.compose.foundation.layout.padding import androidx.compose.foundation.rememberScrollState import androidx.compose.foundation.verticalScroll import androidx.compose.material.icons.Icons import androidx.compose.material.icons.filled.Edit import androidx.compose.material3.AlertDialog import androidx.compose.material3.Button import androidx.compose.material3.Card import androidx.compose.material3.CardDefaults import androidx.compose.material3.ExperimentalMaterial3Api import androidx.compose.material3.FloatingActionButton import androidx.compose.material3.Icon import androidx.compose.material3.MaterialTheme import androidx.compose.material3.OutlinedButton import androidx.compose.material3.Scaffold import androidx.compose.material3.Text import androidx.compose.material3.TextButton import androidx.compose.runtime.Composable import androidx.compose.runtime.getValue import androidx.compose.runtime.mutableStateOf import androidx.compose.runtime.saveable.rememberSaveable import androidx.compose.runtime.setValue import androidx.compose.ui.Modifier import androidx.compose.ui.res.dimensionResource import androidx.compose.ui.res.stringResource import androidx.compose.ui.text.font.FontWeight import androidx.compose.ui.tooling.preview.Preview import com.example.inventory.InventoryTopAppBar import com.example.inventory.R import com.example.inventory.data.Item import com.example.inventory.ui.navigation.NavigationDestination import com.example.inventory.ui.theme.InventoryTheme

object ItemDetailsDestination : NavigationDestination { override val route = "item_details" override val titleRes = R.string.item_detail_title const val itemIdArg = "itemId" val routeWithArgs = "$route/{$itemIdArg}" }

@OptIn(ExperimentalMaterial3Api::class) @Composable fun ItemDetailsScreen( navigateToEditItem: (Int) -> Unit, navigateBack: () -> Unit, modifier: Modifier = Modifier ) { Scaffold( topBar = { InventoryTopAppBar( title = stringResource(ItemDetailsDestination.titleRes), canNavigateBack = true, navigateUp = navigateBack ) }, floatingActionButton = { FloatingActionButton( onClick = { navigateToEditItem(0) }, shape = MaterialTheme.shapes.medium, modifier = Modifier.padding(dimensionResource(id = R.dimen.padding_large))

        ) {
            Icon(
                imageVector = Icons.Default.Edit,
                contentDescription = stringResource(R.string.edit_item_title),
            )
        }
    }, modifier = modifier
) { innerPadding ->
    ItemDetailsBody(
        itemDetailsUiState = ItemDetailsUiState(),
        onSellItem = { },
        onDelete = { },
        modifier = Modifier
            .padding(innerPadding)
            .verticalScroll(rememberScrollState())
    )
}
}

@Composable private fun ItemDetailsBody( itemDetailsUiState: ItemDetailsUiState, onSellItem: () -> Unit, onDelete: () -> Unit, modifier: Modifier = Modifier ) { Column( modifier = modifier.padding(dimensionResource(id = R.dimen.padding_medium)), verticalArrangement = Arrangement.spacedBy(dimensionResource(id = R.dimen.padding_medium)) ) { var deleteConfirmationRequired by rememberSaveable { mutableStateOf(false) }

    ItemDetails(
        item = itemDetailsUiState.itemDetails.toItem(),
        modifier = Modifier.fillMaxWidth()
    )
    Button(
        onClick = onSellItem,
        modifier = Modifier.fillMaxWidth(),
        shape = MaterialTheme.shapes.small,
        enabled = true
    ) {
        Text(stringResource(R.string.sell))
    }
    OutlinedButton(
        onClick = { deleteConfirmationRequired = true },
        shape = MaterialTheme.shapes.small,
        modifier = Modifier.fillMaxWidth()
    ) {
        Text(stringResource(R.string.delete))
    }
    if (deleteConfirmationRequired) {
        DeleteConfirmationDialog(
            onDeleteConfirm = {
                deleteConfirmationRequired = false
                onDelete()
            },
            onDeleteCancel = { deleteConfirmationRequired = false },
            modifier = Modifier.padding(dimensionResource(id = R.dimen.padding_medium))
        )
    }
}
}

@Composable fun ItemDetails( item: Item, modifier: Modifier = Modifier ) { Card( modifier = modifier, colors = CardDefaults.cardColors( containerColor = MaterialTheme.colorScheme.primaryContainer, contentColor = MaterialTheme.colorScheme.onPrimaryContainer ) ) { Column( modifier = Modifier .fillMaxWidth() .padding(dimensionResource(id = R.dimen.padding_medium)), verticalArrangement = Arrangement.spacedBy( dimensionResource(id = R.dimen.padding_medium) ) ) { ItemDetailsRow( labelResID = R.string.item, itemDetail = item.name, modifier = Modifier.padding( horizontal = dimensionResource(id = R.dimen.padding_medium) ) ) ItemDetailsRow( labelResID = R.string.quantity_in_stock, itemDetail = item.quantity.toString(), modifier = Modifier.padding( horizontal = dimensionResource(id = R.dimen.padding_medium) ) ) ItemDetailsRow( labelResID = R.string.price, itemDetail = item.formatedPrice(), modifier = Modifier.padding( horizontal = dimensionResource(id = R.dimen.padding_medium) ) ) } } }

@Composable private fun ItemDetailsRow( @StringRes labelResID: Int, itemDetail: String, modifier: Modifier = Modifier ) { Row(modifier = modifier) { Text(stringResource(labelResID)) Spacer(modifier = Modifier.weight(1f)) Text(text = itemDetail, fontWeight = FontWeight.Bold) } }

@Composable private fun DeleteConfirmationDialog( onDeleteConfirm: () -> Unit, onDeleteCancel: () -> Unit, modifier: Modifier = Modifier ) { AlertDialog(onDismissRequest = { /* Do nothing */ }, title = { Text(stringResource(R.string.attention)) }, text = { Text(stringResource(R.string.delete_question)) }, modifier = modifier, dismissButton = { TextButton(onClick = onDeleteCancel) { Text(stringResource(R.string.no)) } }, confirmButton = { TextButton(onClick = onDeleteConfirm) { Text(stringResource(R.string.yes)) } }) }

@Preview(showBackground = true) @Composable fun ItemDetailsScreenPreview() { InventoryTheme { ItemDetailsBody( ItemDetailsUiState( outOfStock = true, itemDetails = ItemDetails(1, "Pen", "$100", "10") ), onSellItem = {}, onDelete = {} ) } } ITEM EDIT SCREEN package com.example.inventory.ui.item

import androidx.compose.foundation.layout.padding import androidx.compose.material3.ExperimentalMaterial3Api import androidx.compose.material3.Scaffold import androidx.compose.runtime.Composable import androidx.compose.ui.Modifier import androidx.compose.ui.res.stringResource import androidx.compose.ui.tooling.preview.Preview import androidx.lifecycle.viewmodel.compose.viewModel import com.example.inventory.InventoryTopAppBar import com.example.inventory.R import com.example.inventory.ui.AppViewModelProvider import com.example.inventory.ui.navigation.NavigationDestination import com.example.inventory.ui.theme.InventoryTheme

object ItemEditDestination : NavigationDestination { override val route = "item_edit" override val titleRes = R.string.edit_item_title const val itemIdArg = "itemId" val routeWithArgs = "$route/{$itemIdArg}" }

@OptIn(ExperimentalMaterial3Api::class) @Composable fun ItemEditScreen( navigateBack: () -> Unit, onNavigateUp: () -> Unit, modifier: Modifier = Modifier, viewModel: ItemEditViewModel = viewModel(factory = AppViewModelProvider.Factory) ) { Scaffold( topBar = { InventoryTopAppBar( title = stringResource(ItemEditDestination.titleRes), canNavigateBack = true, navigateUp = onNavigateUp ) }, modifier = modifier ) { innerPadding -> ItemEntryBody( itemUiState = viewModel.itemUiState, onItemValueChange = { }, onSaveClick = { }, modifier = Modifier.padding(innerPadding) ) } }

@Preview(showBackground = true) @Composable fun ItemEditScreenPreview() { InventoryTheme { ItemEditScreen(navigateBack = { /Do nothing/ }, onNavigateUp = { /Do nothing/ }) } }

ITEM ENTRY SCREEN package com.example.inventory.ui.item

import androidx.compose.foundation.layout.Arrangement import androidx.compose.foundation.layout.Column import androidx.compose.foundation.layout.fillMaxWidth import androidx.compose.foundation.layout.padding import androidx.compose.foundation.rememberScrollState import androidx.compose.foundation.text.KeyboardOptions import androidx.compose.foundation.verticalScroll import androidx.compose.material3.Button import androidx.compose.material3.ExperimentalMaterial3Api import androidx.compose.material3.MaterialTheme import androidx.compose.material3.OutlinedTextField import androidx.compose.material3.OutlinedTextFieldDefaults import androidx.compose.material3.Scaffold import androidx.compose.material3.Text import androidx.compose.material3.TextFieldDefaults import androidx.compose.runtime.Composable import androidx.compose.ui.Modifier import androidx.compose.ui.res.dimensionResource import androidx.compose.ui.res.stringResource import androidx.compose.ui.text.input.KeyboardType import androidx.compose.ui.tooling.preview.Preview import androidx.lifecycle.viewmodel.compose.viewModel import com.example.inventory.InventoryTopAppBar import com.example.inventory.R import com.example.inventory.ui.AppViewModelProvider import com.example.inventory.ui.navigation.NavigationDestination import com.example.inventory.ui.theme.InventoryTheme import java.util.Currency import java.util.Locale

object ItemEntryDestination : NavigationDestination { override val route = "item_entry" override val titleRes = R.string.item_entry_title }

@OptIn(ExperimentalMaterial3Api::class) @Composable fun ItemEntryScreen( navigateBack: () -> Unit, onNavigateUp: () -> Unit, canNavigateBack: Boolean = true, viewModel: ItemEntryViewModel = viewModel(factory = AppViewModelProvider.Factory) ) { Scaffold( topBar = { InventoryTopAppBar( title = stringResource(ItemEntryDestination.titleRes), canNavigateBack = canNavigateBack, navigateUp = onNavigateUp ) } ) { innerPadding -> ItemEntryBody( itemUiState = viewModel.itemUiState, onItemValueChange = viewModel::updateUiState, onSaveClick = { }, modifier = Modifier .padding(innerPadding) .verticalScroll(rememberScrollState()) .fillMaxWidth() ) } }

@Composable fun ItemEntryBody( itemUiState: ItemUiState, onItemValueChange: (ItemDetails) -> Unit, onSaveClick: () -> Unit, modifier: Modifier = Modifier ) { Column( verticalArrangement = Arrangement.spacedBy(dimensionResource(id = R.dimen.padding_large)), modifier = modifier.padding(dimensionResource(id = R.dimen.padding_medium)) ) { ItemInputForm( itemDetails = itemUiState.itemDetails, onValueChange = onItemValueChange, modifier = Modifier.fillMaxWidth() ) Button( onClick = onSaveClick, enabled = itemUiState.isEntryValid, shape = MaterialTheme.shapes.small, modifier = Modifier.fillMaxWidth() ) { Text(text = stringResource(R.string.save_action)) } } }

@OptIn(ExperimentalMaterial3Api::class) @Composable fun ItemInputForm( itemDetails: ItemDetails, modifier: Modifier = Modifier, onValueChange: (ItemDetails) -> Unit = {}, enabled: Boolean = true ) { Column( modifier = modifier, verticalArrangement = Arrangement.spacedBy(dimensionResource(id = R.dimen.padding_medium)) ) { OutlinedTextField( value = itemDetails.name, onValueChange = { onValueChange(itemDetails.copy(name = it)) }, label = { Text(stringResource(R.string.item_name_req)) }, colors = OutlinedTextFieldDefaults.colors( focusedContainerColor = MaterialTheme.colorScheme.secondaryContainer, unfocusedContainerColor = MaterialTheme.colorScheme.secondaryContainer, disabledContainerColor = MaterialTheme.colorScheme.secondaryContainer, ), modifier = Modifier.fillMaxWidth(), enabled = enabled, singleLine = true ) OutlinedTextField( value = itemDetails.price, onValueChange = { onValueChange(itemDetails.copy(price = it)) }, keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Decimal), label = { Text(stringResource(R.string.item_price_req)) }, colors = OutlinedTextFieldDefaults.colors( focusedContainerColor = MaterialTheme.colorScheme.secondaryContainer, unfocusedContainerColor = MaterialTheme.colorScheme.secondaryContainer, disabledContainerColor = MaterialTheme.colorScheme.secondaryContainer, ), leadingIcon = { Text(Currency.getInstance(Locale.getDefault()).symbol) }, modifier = Modifier.fillMaxWidth(), enabled = enabled, singleLine = true ) OutlinedTextField( value = itemDetails.quantity, onValueChange = { onValueChange(itemDetails.copy(quantity = it)) }, keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Number), label = { Text(stringResource(R.string.quantity_req)) }, colors = OutlinedTextFieldDefaults.colors( focusedContainerColor = MaterialTheme.colorScheme.secondaryContainer, unfocusedContainerColor = MaterialTheme.colorScheme.secondaryContainer, disabledContainerColor = MaterialTheme.colorScheme.secondaryContainer, ), modifier = Modifier.fillMaxWidth(), enabled = enabled, singleLine = true ) if (enabled) { Text( text = stringResource(R.string.required_fields), modifier = Modifier.padding(start = dimensionResource(id = R.dimen.padding_medium)) ) } } }

@Preview(showBackground = true) @Composable private fun ItemEntryScreenPreview() { InventoryTheme { ItemEntryBody(itemUiState = ItemUiState( ItemDetails( name = "Item name", price = "10.00", quantity = "5" ) ), onItemValueChange = {}, onSaveClick = {}) } }

ITEM VIEW MODEL

package com.example.inventory.ui.item

import androidx.compose.runtime.getValue import androidx.compose.runtime.mutableStateOf import androidx.compose.runtime.setValue import androidx.lifecycle.ViewModel import com.example.inventory.data.Item import java.text.NumberFormat

/**

ViewModel to validate and insert items in the Room database. */ class ItemEntryViewModel : ViewModel() {

/**

Holds current item ui state */ var itemUiState by mutableStateOf(ItemUiState()) private set
/**

Updates the [itemUiState] with the value provided in the argument. This method also triggers
a validation for input values. */ fun updateUiState(itemDetails: ItemDetails) { itemUiState = ItemUiState(itemDetails = itemDetails, isEntryValid = validateInput(itemDetails)) }
private fun validateInput(uiState: ItemDetails = itemUiState.itemDetails): Boolean { return with(uiState) { name.isNotBlank() && price.isNotBlank() && quantity.isNotBlank() } } }

/**

Represents Ui State for an Item. */ data class ItemUiState( val itemDetails: ItemDetails = ItemDetails(), val isEntryValid: Boolean = false )
data class ItemDetails( val id: Int = 0, val name: String = "", val price: String = "", val quantity: String = "", )

/**

Extension function to convert [ItemDetails] to [Item]. If the value of [ItemDetails.price] is
not a valid [Double], then the price will be set to 0.0. Similarly if the value of
[ItemDetails.quantity] is not a valid [Int], then the quantity will be set to 0 */ fun ItemDetails.toItem(): Item = Item( id = id, name = name, price = price.toDoubleOrNull() ?: 0.0, quantity = quantity.toIntOrNull() ?: 0 )
fun Item.formatedPrice(): String { return NumberFormat.getCurrencyInstance().format(price) }

/**

Extension function to convert [Item] to [ItemUiState] */ fun Item.toItemUiState(isEntryValid: Boolean = false): ItemUiState = ItemUiState( itemDetails = this.toItemDetails(), isEntryValid = isEntryValid )
/**

Extension function to convert [Item] to [ItemDetails] */ fun Item.toItemDetails(): ItemDetails = ItemDetails( id = id, name = name, price = price.toString(), quantity = quantity.toString() )
INVENTORYAPP
@file:OptIn(ExperimentalMaterial3Api::class)
package com.example.inventory

import android.annotation.SuppressLint import androidx.compose.material.icons.Icons.Filled import androidx.compose.material.icons.filled.ArrowBack import androidx.compose.material3.CenterAlignedTopAppBar import androidx.compose.material3.ExperimentalMaterial3Api import androidx.compose.material3.Icon import androidx.compose.material3.IconButton import androidx.compose.material3.Text import androidx.compose.material3.TopAppBarScrollBehavior import androidx.compose.runtime.Composable import androidx.compose.ui.Modifier import androidx.compose.ui.res.stringResource import androidx.navigation.NavHostController import androidx.navigation.compose.rememberNavController import com.example.inventory.R.string import com.example.inventory.ui.navigation.InventoryNavHost

/**

Top level composable that represents screens for the application. */ @SuppressLint("UnusedMaterial3ScaffoldPaddingParameter") @Composable fun InventoryApp(navController: NavHostController = rememberNavController()) { InventoryNavHost(navController = navController) }
/**

App bar to display title and conditionally display the back navigation. */ @Composable fun InventoryTopAppBar( title: String, canNavigateBack: Boolean, modifier: Modifier = Modifier, scrollBehavior: TopAppBarScrollBehavior? = null, navigateUp: () -> Unit = {} ) { CenterAlignedTopAppBar( title = { Text(title) }, modifier = modifier, scrollBehavior = scrollBehavior, navigationIcon = { if (canNavigateBack) { IconButton(onClick = navigateUp) { Icon( imageVector = Filled.ArrowBack, contentDescription = stringResource(string.back_button) ) } } } ) } INVENTORY APPLICATION
package com.example.inventory

import android.app.Application import com.example.inventory.data.AppContainer import com.example.inventory.data.AppDataContainer

class InventoryApplication : Application() {

/**
 * AppContainer instance used by the rest of classes to obtain dependencies
 */
lateinit var container: AppContainer

override fun onCreate() {
    super.onCreate()
    container = AppDataContainer(this)
}
} MAIN ACTIVITY package com.example.inventory

import android.os.Bundle import androidx.activity.ComponentActivity import androidx.activity.compose.setContent import androidx.compose.foundation.layout.fillMaxSize import androidx.compose.material3.MaterialTheme import androidx.compose.material3.Surface import androidx.compose.ui.Modifier import com.example.inventory.ui.theme.InventoryTheme

class MainActivity : ComponentActivity() { override fun onCreate(savedInstanceState: Bundle?) { super.onCreate(savedInstanceState) setContent { InventoryTheme { // A surface container using the 'background' color from the theme Surface( modifier = Modifier.fillMaxSize(), color = MaterialTheme.colorScheme.background ) { InventoryApp() } } } } }
