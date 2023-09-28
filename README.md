# EnhancedDataTables

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
    * [Context Menu](#context-menu)
* [Sample](#sample)
* [License](#license)
* [Author](#Author)
* [History](#History)

## Description

This plugin adds multiple data tables with extended functionality, including a data table that allows you to add development-only rows that are not included in shipping and test builds, and a data table that generates enum-compliant rows.  

## Requirement

Target version : UE5.0 ~ 5.3  
Target platform :  Windows, Mac, Linux (Runtime module has no platform restrictions)

## Installation

IN-PREPARATION

## Features And Usages

### Development Data Table

`Development Data Table` is an extended data table asset that allows you to add development-only row data that is deleted during shipping builds and test builds.


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

|     *Name*      | *Description*                                                                                                                               |
|:---------------:|:--------------------------------------------------------------------------------------------------------------------------------------------|
| ExposeOnRowName | Adding this specifier to an element with the Hidden specifier will make it visible on the `Enum Based Data Table` editor.                   |
| HiddenOnRowName | Adding this specifier to an element that does not have a Hidden specifier will cause it to be hidden on the `Enum Based Data Table` editor. |
| DevelopmentOnly | If this specifier is added, the corresponding row data will not be cooked in the shipping and test builds.                                  |


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

### Context Menu

![DataTableContextMenu](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/6e3c5caa-0f75-4593-9377-653a663d86fe)
![DevelopmentDataTableContextMenu](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/a1b5390c-ac46-4605-8749-02f01bf5f6c5)
![EnumBasedDataTableContextMenu](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/80032f5c-db78-4e9f-882d-6308cc6b192a)
![GroupedDataTableContextMenu](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/ed61ea5c-14dd-4b5a-9f3d-db25cb6a1887)

You can use the conversion function between each data table from the context menu of the regular data table asset and the extended data table asset added with this plugin.  
Also, you can use the function to replace the original enumeration in `Enum Based Data Table` context menu.  

## Sample

You can download sample project development and shipping packages so that you can check the operation before purchasing.  
You can also download images and source code that allow you to see Blueprint graphs and data tables so that you can see the data in the editor of the sample project.  

[Download Sample](https://github.com/Naotsun19B/EnhancedDataTables-Document/releases/tag/Sample)

## License

[MIT License](https://en.wikipedia.org/wiki/MIT_License)

## Author

[Naotsun](https://twitter.com/Naotsun_UE)

## History

IN-PREPARATION
