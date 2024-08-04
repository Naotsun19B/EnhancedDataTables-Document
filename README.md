# EnhancedDataTables

![Plugins](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/bf98dec5-e403-491d-bb52-e4ec8cb21113)

<!--ts-->
* [Description](#description)
* [Requirement](#requirement)
* [Installation](#installation)
* [Features And Usages](#features-and-usages)
    * [Development Data Table](#development-data-table)
        * [Asset](#asset)
        * [Editor](#editor)
        * [Nodes](#nodes)
    * [Enum Based Data Table](#enum-based-data-table)
        * [Asset](#asset)
        * [Editor](#editor)
        * [Nodes](#nodes)
    * [Grouped Data Table](#grouped-data-table)
        * [Asset](#asset)
        * [Editor](#editor)
        * [Nodes](#nodes)
        * [Data Table Group Mapper](#data-table-group-mapper)
    * [Data Table Row Coloration](#data-table-row-coloration)
      * [Data Table Row Coloration implemented in plugin standard](#data-table-row-coloration-implemented-in-plugin-standard)
      * [Implementation in C++](#implementation-in-c)
      * [Implementation in Blueprint](#implementation-in-blueprint)
    * [Context Menu](#context-menu)
    * [Import Options](#import-options)
* [Sample](#sample)
* [License](#license)
* [Author](#Author)
* [History](#History)

## Description

This plugin adds multiple data tables with extended functionality, including a data table that allows you to add development-only rows that are not included in shipping and test builds, and a data table that generates enum-compliant rows.  
Additionally, a feature will be added that allows users to arbitrarily determine the background and text color of each row in the editor.  


## Requirement

Target version : UE5.0 ~ 5.4  
Target platform :  Windows, Mac, Linux (Runtime module has no platform restrictions)


## Installation

Install from the [marketplace](https://www.unrealengine.com/marketplace/en-US/product/20e3da7acc924ed1b0f018412a443c9a).  
If the feature is not available after installing the plugin, it is possible that the plugin has not been enabled, so please check if the plugin is enabled from Edit > Plugins.


## Features And Usages

### Development Data Table

`Development Data Table` is an extended data table asset that allows you to add development-only row data that is deleted during shipping builds and test builds.  
You can set build configurations that removes development-only row data from project settings.

![DevelopmentDataTableSettings](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/ee19b660-1505-4809-bfb7-0c437fc67997)


#### ・Asset

![CreateDevelopmentDataTable](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/f697730d-ee51-4e62-8d45-f0282b9e9120)

You can create assets from Miscellaneous in the context menu of the content browser.  
Here you can select a structure inheriting from `FDevelopmentDataTableRowBase` defined in C++.  
Alternatively, you can only select structure assets that have a variable of type `bool` defined, `bIsDevelopmentOnly`.  

```DevelopmentDataTableRow.h
USTRUCT(BlueprintType)
struct FTestDevelopmentDataTableRow : public FDevelopmentDataTableRowBase
{
	GENERATED_BODY()

public:
	UPROPERTY(EditAnywhere, BlueprintReadOnly)
	int32 TestInt = 5;

	UPROPERTY(EditAnywhere, BlueprintReadOnly)
	FString TestString;

	UPROPERTY(EditAnywhere, BlueprintReadOnly)
	bool bTestBool = false;

	UPROPERTY(EditAnywhere, BlueprintReadOnly)
	TSubclassOf<UObject> TestClass;
};
```

![S_TestDevelopmentDataTableRow](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/f78ac5b4-418a-4431-bf4f-20de9b51efa6)


#### ・Editor

![DevelopmentDataTableEditor](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/80f4c736-7b8b-4cce-8614-e08ec8682071)

The asset editor has also been enhanced, and lines with the development-only flag set to `true' have a background that changes to indicate development-only.  
You can change whether a row is development-only or not from the details panel on the editor, just like any other row property.  


#### ・Nodes

![IsDevelopmentOnly](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/a67d8185-84c3-48f7-ae9c-40f293c420ff)

You can get whether the specified row of `Development Data Table` is a development-only row.


### Enum Based Data Table

`Enum Based Data Table` is an extended data table asset based on an enumeration that specifies the enumeration that is the source of the row name and order in addition to the row structure.


#### ・Asset

![CreateEnumBasedDataTable](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/813edae0-d365-4bde-9f2c-1c1e74259b91)

You can create assets from Miscellaneous in the context menu of the content browser.  
Here you can select a structure inheriting from `FEnumBasedDataTableRowBase` defined in C++.  
Alternatively, as with `Development Data Table`, you can only select structure assets that have a variable of type `bool` defined, `bIsDevelopmentOnly`.  
Similarly, during creation, you select the enumeration that will be the source of the row name and order, but here you can select both enumerations defined in C++ and enumeration assets.

```EnumBasedDataTableEnumAndRow.h
UENUM(BlueprintType)
enum class ETestEnumBasedDataTableSource : uint8
{
	ItemA,
	ItemB UMETA(Hidden, ExposeOnRowName),
	ItemC,
	ItemD UMETA(HiddenOnRowName),
	ItemE,
	ItemF UMETA(DevelopmentOnly),
	ItemG,
	ItemH UMETA(Hidden),
};

USTRUCT(BlueprintType)
struct FTestEnumBasedDataTableRow : public FEnumBasedDataTableRowBase
{
	GENERATED_BODY()

public:
	UPROPERTY(EditAnywhere, BlueprintReadOnly)
	FName TestName;

	UPROPERTY(EditAnywhere, BlueprintReadOnly)
	uint8 TestByte = 255;
};
```

As shown in the sample code above, several meta specifiers are available for the enumeration that forms the basis of `Enum Based Data Table`.  

|         *Name*         | *Description*                                                                                                                               |
|:----------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------|
|    ExposeOnRowName     | Adding this specifier to an element with the Hidden specifier will make it visible on the `Enum Based Data Table` editor.                   |
|    HiddenOnRowName     | Adding this specifier to an element that does not have a Hidden specifier will cause it to be hidden on the `Enum Based Data Table` editor. |
|    DevelopmentOnly     | If this specifier is added, the corresponding row data will not be cooked in the shipping and test builds.                                  |
| RowBackgroundTintColor | Specifies the background color of the corresponding row on the editor used by the `Data Table Row Coloration` function described later.     |
|      RowTextColor      | Specifies the text color of the corresponding row on the editor used by the `Data Table Row Coloration` function described later.           |


The following formats can be used when specifying colors with `RowBackgroundTintColor` and `RowTextColor`:  

- Color name defined in `FLinearColor`. (White, Gray, Black, Transparent, Red, Green, Blue, Yellow)  
- Hexadecimal color code starting with `#`.  
- Comma-separated integer values arranged in "R,G,B,A" format.  
- Comma-separated floating values arranged in "R,G,B,A" format.  

If comma separated format, alpha is optional and can be ordered in any order by prepending RGBA= to the value.  


#### ・Editor

![EnumBasedDataTableEditor](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/fca04d30-c638-4223-8e81-f17d8b0d2bda)

The asset editor has also been enhanced, similar to the `Development Data Table`, where rows with the development-only flag set to `true` have a background that changes to indicate development-only.  
Additionally, adding, deleting, and moving rows cannot be done from the editor because they are based on the original enumeration.    
Development-only flags cannot be changed from the editor if the underlying enumeration is defined in C++.  
If the based enumeration is an asset, it can be changed from the detail panel as in the `Development Data Table` asset editor.  


#### ・Nodes

![IsValidRowName](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/74dc20f2-1618-4cdf-9cbf-3446a408ce98)

You can get whether the specified row name is included in the enumeration that is the source of `Enum Based Data Table`.

![HasRowEnumDevelopmentOnly](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/b141e38c-9448-4717-83a4-aece71c3c4ac)

You can get whether the specified row name has the `DevelopmentOnly` meta specifier in the enumeration that is the source of `Enum Based Data Table`.

![GetEnumBasedDataTableRowFromEnum](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/4283cc24-87c4-46fa-803d-1dbe9897e80c)
![GetEnumBasedDataTableRowFromEnum_Assigned](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/60958eda-f722-4c68-82cd-efff3d673321)

You can retrieve row data from the `Enum Based Data Table` by specifying the row enumeration.  
If you specify an asset directly to the `Enum Based Data Table` pin, the types of the `Row Enum` and `Out Row` pins will be linked.  


### Grouped Data Table

`Grouped Data Table` is an extended data table asset that allows you to obtain the names and data of multiple rows with the same value based on the `Group Id` included in the row structure.  


#### ・Asset

![CreateGroupedDataTable](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/ad47d4c0-b4b9-4f1e-bd73-3dbd89def155)

You can create assets from Miscellaneous in the context menu of the content browser.  
Similar to `Development Data Table`, here you can select a structure inheriting from `FDevelopmentDataTableRowBase` defined in C++.  
Alternatively, you can only select structure assets that have a variable of type `bool` defined, `bIsDevelopmentOnly`.  
Both structures and structure assets defined in C++ must have a property named `Group Id` of the corresponding type defined.  

```GroupedDataTableRow.h
USTRUCT(BlueprintType)
struct FTestGroupedDataTableRow_Name : public FDevelopmentDataTableRowBase
{
	GENERATED_BODY()

public:
	UPROPERTY(EditAnywhere, BlueprintReadOnly)
	FName GroupId;

	UPROPERTY(EditAnywhere, BlueprintReadOnly)
	int32 TestInt = 9;
};
```

![S_TestGroupedDataTableRow](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/60916e4a-369d-49a1-a8f6-c94f668d2961)

The following types can be used for `Group Id`. The type name in C++ and the type name in parentheses are the type name in Blueprint.

`bool (Boolean)`、`uint8 (Byte)`、`int32 (Integer)`、`int64 (Integer64)`、`float (Flout)`、`double`、`FName (Name)`、`FString (String)`

In addition to these, you can also use enums (`Enum`) defined by C++ and enumeration type assets, and Pulldown structures added by the [Pulldown Builder](https://github.com/Naotsun19B/PulldownBuilder/blob/master/README.md).


#### ・Editor

![GroupedDataTableEditor](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/fe3fd079-4d7c-443d-ba94-170f61fb3fb7)

The asset editor has also been enhanced, similar to the `Development Data Table`, where rows with the development-only flag set to `true` have a background that changes to indicate development-only.  
Also, `Group Id` is editable like any other property.  


#### ・Nodes

![GetGroupedDataTableRowsFromGroupId](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/4da94398-40fa-45fb-a294-f7eb3eead604)
![GetGroupedDataTableRowsFromGroupId_Assigned](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/25812f27-dc03-45b6-b919-e9871ac7ff71)

You can specify `Group Id` to retrieve data for multiple rows included in that group from `Grouped Data Table`.  
If you specify an asset directly to the `Grouped Data Table` pin, the types of the `Group Id` and `Out Rows` pins will be linked.  


#### ・Data Table Group Mapper

Although it requires the use of C++, there is a utility class template that allows data tables of types other than `Grouped Data Table` to be treated like `Group Id` values for certain properties and data to be retrieved in batches.  

```DataTableGroupMapper.h

#include "GroupedDataTable/Utilities/DataTableGroupMapper.h"

USTRUCT(BlueprintType)
struct FTestDataTableRow : public FTableRowBase
{
	GENERATED_BODY()

public:
	UPROPERTY(EditAnywhere, BlueprintReadOnly)
	FString TestString;

	UPROPERTY(EditAnywhere, BlueprintReadOnly)
	uint8 TestByte = 3;
};

UCLASS()
class UTestFunctions : public UBlueprintFunctionLibrary
{
	GENERATED_BODY()

public:
	UFUNCTION(BlueprintCallable)
	static void TestDataTableGroupMapper(const UDataTable* DataTable, const uint8 TestByte);
};
```

```DataTableGroupMapper.cpp
void UTestFunctions::TestDataTableGroupMapper(const UDataTable* DataTable, const uint8 TestByte)
{
	const auto DataTableGroupMapper = MakeShared<EnhancedDataTables::TDataTableGroupMapper<uint8, FTestDataTableRow>>(DataTable, GET_MEMBER_NAME_CHECKED(FTestDataTableRow, TestByte));
	const TArray<FName>& MultipleRowNameInGroup = DataTableGroupMapper->GetMultipleRowNameInGroup(TestByte);
	const TArray<const FTestDataTableRow*>& MultipleRowDataInGroup = DataTableGroupMapper->GetMultipleRowDataInGroup(TestByte);
	// Do something.
}
```

There are more types that can be used as `Group Id` of `TDataTableGroupMapper` than types that can be used as `Group Id` of `Grouped Data Table`, and they can be used if the following conditions are met.  
・Must not be an array type, map type, or pointer type  
・The types can be compared (`==` operator can be used)   
・An overload of LexFromString is defined  

Also available is `TIsGroupId`, a trait class that determines whether a given type can be used as a `Group Id`.  


### Data Table Row Coloration

This is a function that allows you to determine the background color and text color of the rows on the editor of the extended data table added by this plugin based on arbitrary conditions such as row name, row data, row number, etc.  
Also, when using C++, it is also possible to change the image used for the background of the row.   
To change the background color or text color, you need to create a class that inherits from `UDataTableRowColoration` in C++ or Blueprint and implement the conditions.   

![Details-DDT_UserDefinedStruct](https://github.com/user-attachments/assets/1242ab85-a6f4-44ef-980a-39125e3bbb4d)

You can apply `UDataTableRowColoration`, which is implemented in the plugin standard or implemented by user, by opening the details panel of the data table editor and setting it to `Row Coloration Class`.


#### Data Table Row Coloration implemented in plugin standard

|                *Name*                 |           C++ Class Name           | *Description*                                                                                                                                                                    |
|:-------------------------------------:|:----------------------------------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Development Data Table Row Coloration | UDevelopmentDataTableRowColoration | Class set as standard in `Development Data Table`. <br>Changes the background image and text color of rows only during development.                                              |
| Enum Based Data Table Row Coloration  |  UEnumBasedDataTableRowColoration  | Class set as standard for `Enum Based Data Table`. <br>Enables changing the background tint and text color from the meta specifier of the underlying enumeration defined in C++. |


#### Implementation in C++

Please inherit `UDataTableRowColoration` or the class implemented in the plug-in standard and override the functions below according to your needs.  

|      *Function Name*      | *Description*                                   |
|:-------------------------:|:------------------------------------------------|
|   Get Background Brush    | Returns the background image brush for the row. |
| Get Background Tint Color | Returns the background tint color for the row.  |      
|      Get Text Color       | Returns the text color for the row.             |

Below is the implementation of the background tint color of `UDevelopmentDataTableRowColoration`.  

```DevelopmentDataTableRowColoration.cpp
FSlateColor UDevelopmentDataTableRowColoration::GetBackgroundTintColor_Implementation(
	const UDataTable* DataTable,
	const FName& RowName,
	const FInstancedStruct& RowData,
	const int32 RowIndex
) const
{
	check(IsValid(DataTable));
	
	const bool bIsDevelopmentOnly = UDevelopmentDataTableFunctionLibrary::IsDevelopmentOnly(Cast<UDevelopmentDataTable>(DataTable), RowName);
	if (!bIsDevelopmentOnly)
	{
		return Super::GetBackgroundTintColor_Implementation(DataTable, RowName, RowData, RowIndex);
	}

	const bool bIsOddRow = IsOddRow(RowIndex);
	return (bIsOddRow ? OddDevelopmentOnlyBackgroundTintColor : EvenDevelopmentOnlyBackgroundTintColor);
}
```

#### Implementation in Blueprint

Create a Blueprint class that inherits `Data Table Row Coloration` or a class implemented in the plugin standard.  

![DataTableRowColoration_Simple](https://github.com/user-attachments/assets/6ac94b07-602d-4176-9279-fe37f6fc3415)

You can create a Blueprint class that inherits the class implemented in the plug-in standard and change the properties to change the color.  

![DataTableRowColoration_Complex](https://github.com/user-attachments/assets/bc01a31c-7caa-4c32-ab4f-275d5546d43e)
![BP_GroupedDataTableRowColoration-GetBackgroundTintColor](https://github.com/user-attachments/assets/a99b8291-0e2a-4d9a-b1a1-afd49bc9eb2a)

You can also override and implement the functions below according to your needs in the Blueprint class you created.  
In the image above, the colors are changed for each type of `Group Id` in the `Grouped Data Table`.  

|      *Function Name*      | *Description*                                   |
|:-------------------------:|:------------------------------------------------|
| Get Background Tint Color | Returns the background tint color for the row.  |      
|      Get Text Color       | Returns the text color for the row.             |

The argument `Raw Data` is of `Instanced Struct` type, so use the `Get Instanced Struct Value` function to convert it to any row structure type.  


### Context Menu

![DataTableContextMenu](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/6e3c5caa-0f75-4593-9377-653a663d86fe)
![DevelopmentDataTableContextMenu](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/a1b5390c-ac46-4605-8749-02f01bf5f6c5)
![EnumBasedDataTableContextMenu](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/80032f5c-db78-4e9f-882d-6308cc6b192a)
![GroupedDataTableContextMenu](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/ed61ea5c-14dd-4b5a-9f3d-db25cb6a1887)

You can use the conversion function between each data table from the context menu of the regular data table asset and the extended data table asset added with this plugin.  
Also, you can use the function to replace the original enumeration in `Enum Based Data Table` context menu.  


### Import Options

![DataTableOptions_Original](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/a581d47c-94db-4a90-a514-933eb44b12f9)

Since the engine's standard import option from json or csv does not allow you to select the extended data table added by this plugin, we have added an import option with similar functionality.  

![DataTableOptions_Idle](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/fa18a5b7-16ad-4f3c-a710-20b4fbde6c2c)
![DataTableOptions_Menu](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/9d17b54e-18e6-4015-b9bf-1dd1541e5666)
![DataTableOptions_Selected](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/0fd88a94-9468-4f02-9036-028b3443e7aa)

Unlike the engine standard ones, structures and enumerations used in data tables can be selected after pressing the apply button, similar to the procedure for creating new assets.  
Also, this can only be selected using engine standard import options such as curve tables, but if you press the Cancel button, the engine standard import options will be displayed, so please use them.  


## Sample

You can download sample project development and shipping packages so that you can check the operation before purchasing.  
You can also download images and source code that allow you to see Blueprint graphs and data tables so that you can see the data in the editor of the sample project.  

[Download Sample](https://github.com/Naotsun19B/EnhancedDataTables-Document/releases/tag/Sample)


## License

[MIT License](https://en.wikipedia.org/wiki/MIT_License)


## Author

[Naotsun](https://twitter.com/Naotsun_UE)


## History

- (2024/08/04) v1.7  
  Added `Data Table Row Coloration` that allows users to decide the background and text color of each row in the extended data table asset editor using arbitrary conditions.  

- (2024/08/03) v1.6  
  Fixed a bug that would cause a crash if changes were made to the structure asset that is the source of the enhanced data table asset while the editor of the enhanced data table asset was open

- (2024/06/25) v1.5  
  Fixed an issue where development-only data was not deleted when creating a package from the project launcher  

- (2024/04/24) v1.4  
  Added support for UE5.4  
  Fixed a bug that caused a crash when playing standalone  

- (2024/03/09) v1.3  
  Fixed an issue where import options were displayed even when re-importing  
  Fixed an issue where data added using the duplicate button would disappear when the editor was closed  
  Fixed an issue where the export and open source file items were not displayed in the asset context menu in UE 5.1 and earlier versions  
  Fixed an issue where source files could not be set from the asset editor  
  Fixed an issue where changing the position of already existing row data and restarting the editor would reset the changes  
  Added group id property type to the asset registry tag of `Grouped Data Table`  

- (2023/12/16) v1.2   
  Added ability to change build configuration from project settings that requires deleting development-only rows  
  Fixed an issue where the development overlay would shift position when scrolling in the line editor  

- (2023/11/07) v1.1   
  Fixed an issue where the internal information of `Grouped Data Table` was not updated properly  
  Added the ability to import extended data tables from json or csv  
  Fixed so that re-import can be done with extended data table

- (2023/11/01) v1.0   
  Publish plugin
