# EnhancedDataTables

![Plugins](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/bf98dec5-e403-491d-bb52-e4ec8cb21113)

<!--ts-->
* [概要](#概要)
* [動作環境](#動作環境)
* [インストール](#インストール)
* [機能と使い方](#機能と使い方)
  * [Development Data Table](#development-data-table)
    * [アセット](#アセット)
    * [エディタ](#エディタ)
    * [ノード](#ノード)
  * [Enum Based Data Table](#enum-based-data-table)
    * [アセット](#アセット)
    * [エディタ](#エディタ) 
    * [ノード](#ノード)
  * [Grouped Data Table](#grouped-data-table)
    * [アセット](#アセット)
    * [エディタ](#エディタ)
    * [ノード](#ノード)
    * [Data Table Group Mapper](#data-table-group-mapper)
  * [コンテキストメニュー](#コンテキストメニュー)
  * [インポートオプション](#インポートオプション)
* [サンプル](#サンプル)
* [ライセンス](#ライセンス)
* [作者](#作者)
* [履歴](#履歴)

## 概要

このプラグインは拡張機能を備えた複数のデータテーブルを追加します。  
出荷ビルドやテストビルドには含まれない開発専用の行を追加できるデータテーブルや、列挙体に基づいた行を生成するデータテーブルなどが含まれます。  

## 動作環境

対象バージョン : UE5.0 ~ 5.4    
対象プラットフォーム : Windows, Mac, Linux (ランタイムモジュールはプラットフォームの制限無し)

## インストール

[マーケットプレイス](https://www.unrealengine.com/marketplace/ja/product/20e3da7acc924ed1b0f018412a443c9a)からインストールしてください。  
プラグインのインストール後に機能が使用できない場合は、プラグインが有効化されていない可能性がありますので、編集 > プラグイン からプラグインの有効のチェックが入っているかをご確認ください。

## 機能と使い方

### Development Data Table

`Development Data Table`はデフォルトでシッピングビルドとテストビルドの際に削除される開発時専用の行データを追加することができる拡張データテーブルアセットです。  
プロジェクト設定から開発専用の行データを削除するビルド構成を設定することができます。  

![DevelopmentDataTableSettings](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/ee19b660-1505-4809-bfb7-0c437fc67997)

#### ・アセット

![CreateDevelopmentDataTable](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/f697730d-ee51-4e62-8d45-f0282b9e9120)

コンテンツブラウザのコンテキストメニューのその他からアセットを作成することができます。  
作成時に行の構造体を選択しますが、ここではC++で定義された`FDevelopmentDataTableRowBase`を継承した構造体か、`bool`型の`bIsDevelopmentOnly`という名前の変数が定義されている構造体アセットのみが選択可能です。  

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


#### ・エディタ

![DevelopmentDataTableEditor](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/80f4c736-7b8b-4cce-8614-e08ec8682071)

アセットエディタも拡張されていて、開発専用フラグが`true`の行は背景が開発専用を示す模様に変わります。  
開発専用の行かどうかは他の行のプロパティと同様に、エディタ上の詳細パネルから変更できます。  


#### ・ノード

![IsDevelopmentOnly](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/a67d8185-84c3-48f7-ae9c-40f293c420ff)

`Development Data Table`の指定した行が開発専用の行かどうかを取得することができます。


### Enum Based Data Table

`Enum Based Data Table`は行の構造体とは別に行名や順序の元となる列挙体を指定する列挙体をベースとした拡張データテーブルアセットです。  


#### ・アセット

![CreateEnumBasedDataTable](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/813edae0-d365-4bde-9f2c-1c1e74259b91)

コンテンツブラウザのコンテキストメニューのその他からアセットを作成することができます。  
作成時に行の構造体を選択しますが、ここではC++で定義された`FEnumBasedDataTableRowBase`を継承した構造体か、`Development Data Table`と同様`bool`型の`bIsDevelopmentOnly`という名前の変数が定義されている構造体アセットのみが選択可能です。  
同じく作成時に行名や順序の元となる列挙体を選択しますが、ここではC++で定義された列挙体と列挙体アセットの両方が選択可能です。  

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

上記のサンプルコードにもある通り、`Enum Based Data Table`の元となる列挙体ではいくつかのメタ指定子が利用可能です。  

|      *名前*       | *説明*                                                                         |
|:---------------:|:-----------------------------------------------------------------------------|
| ExposeOnRowName | Hidden 指定子が指定されている要素にこの指定子を追加すると、`Enum Based Data Table`エディタ上では表示されるようになります。 |
| HiddenOnRowName | Hidden 指定子を持たない要素にこの指定子を追加すると、`Enum Based Data Table`エディタ上では非表示になります。        |
| DevelopmentOnly | この指定子を追加すると、対応する行データはシッピングビルドおよびテストビルドではクックされません。                            |


#### ・エディタ

![EnumBasedDataTableEditor](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/fca04d30-c638-4223-8e81-f17d8b0d2bda)

アセットエディタも拡張されていて、`Development Data Table`と同様で開発専用フラグが`true`の行は背景が開発専用を示す模様に変わります。  
また、行の追加や削除、移動などは元となる列挙体をベースとするためエディタ上から操作することはできません。  
開発専用フラグは元となる列挙体がC++で定義されている場合はエディタ上から変更ません。  
元となる列挙体がアセットの場合は`Development Data Table`のアセットエディタと同様に詳細パネルから変更可能です。  


#### ・ノード

![IsValidRowName](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/74dc20f2-1618-4cdf-9cbf-3446a408ce98)

指定した行名が`Enum Based Data Table`の元となる列挙体に含まれているかどうかを取得することができます。  

![HasRowEnumDevelopmentOnly](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/b141e38c-9448-4717-83a4-aece71c3c4ac)

指定した行名が`Enum Based Data Table`の元となる列挙体で`DevelopmentOnly`のメタ指定子を持っているかどうかを取得することができます。  

![GetEnumBasedDataTableRowFromEnum](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/4283cc24-87c4-46fa-803d-1dbe9897e80c)
![GetEnumBasedDataTableRowFromEnum_Assigned](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/60958eda-f722-4c68-82cd-efff3d673321)

行の列挙体を指定して`Enum Based Data Table`から行のデータを取得することができます。  
`Enum Based Data Table`ピンに直接アセットを指定すると`Row Enum`ピンと`Out Row`ピンの型が紐づいたものに変わります。  


### Grouped Data Table

`Grouped Data Table`は行の構造体に含まれる`Group Id`を元に同じ値の複数の行の名前やデータを取得することができる拡張データテーブルアセットです。  

#### ・アセット  

![CreateGroupedDataTable](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/ad47d4c0-b4b9-4f1e-bd73-3dbd89def155)

コンテンツブラウザのコンテキストメニューのその他からアセットを作成することができます。  
作成時に行の構造体を選択しますが、ここでは`Development Data Table`と同様でC++で定義された`FDevelopmentDataTableRowBase`を継承した構造体か、`bool`型の`bIsDevelopmentOnly`という名前の変数が定義されている構造体アセットで、対応した型の`GroupId`という名前のプロパティが定義されている物のみが選択可能です。  

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

`Group Id`の型として使用できるのは下記の通りです。C++での型名とカッコの中がBlueprintでの型名です。  

`bool (Boolean)`、`uint8 (Byte)`、`int32 (Integer)`、`int64 (Integer64)`、`float (Flout)`、`double`、`FName (Name)`、`FString (String)`  

また、この他にもC++や列挙型アセットによって定義された列挙型(`Enum`)や[Pulldown Builder](https://github.com/Naotsun19B/PulldownBuilder/blob/master/README_JP.md)で追加されるPulldown構造体も使用できます。  


#### ・エディタ  

![GroupedDataTableEditor](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/fe3fd079-4d7c-443d-ba94-170f61fb3fb7)

アセットエディタも拡張されていて、`Development Data Table`と同様で開発専用フラグが`true`の行は背景が開発専用を示す模様に変わります。  
また、`Group Id`は他のプロパティと同様に編集可能です。  


#### ・ノード  

![GetGroupedDataTableRowsFromGroupId](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/4da94398-40fa-45fb-a294-f7eb3eead604)
![GetGroupedDataTableRowsFromGroupId_Assigned](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/25812f27-dc03-45b6-b919-e9871ac7ff71)


`Group Id`を指定して`Grouped Data Table`からそのグループに含まれる複数の行のデータを取得することができます。  
`Grouped Data Table`ピンに直接アセットを指定すると`Group Id`ピンと`Out Rows`ピンの型が紐づいたものに変わります。  

#### ・Data Table Group Mapper

C++を使用する必要がありますが、`Grouped Data Table`以外の種類のデータテーブルでも特定のプロパティの値を`Group Id`のように扱い、一括でデータを取得できるようにするユーティリティクラステンプレートがあります。  

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

`TDataTableGroupMapper`の`Group Id`として使用できる型は`Grouped Data Table`の`Group Id`として使用できる型よりも多く、下記の条件を満たしていれば使用可能です。  
・配列型、マップ型、ポインタ型でないこと  
・その型同士で比較ができること（`==`演算子が使用可能）  
・LexFromStringのオーバロードが定義されていること  

また、指定した型が`Group Id`として使用できるかどうかを判定する特性クラスである`TIsGroupId`も利用できます。  

### コンテキストメニュー

![DataTableContextMenu](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/6e3c5caa-0f75-4593-9377-653a663d86fe)
![DevelopmentDataTableContextMenu](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/a1b5390c-ac46-4605-8749-02f01bf5f6c5)
![EnumBasedDataTableContextMenu](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/80032f5c-db78-4e9f-882d-6308cc6b192a)
![GroupedDataTableContextMenu](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/ed61ea5c-14dd-4b5a-9f3d-db25cb6a1887)

通常のデータテーブルアセットとこのプラグインで追加される拡張データテーブルアセットのコンテキストメニューから各データテーブル間の変換機能が利用できます。  
また、`Enum Based Data Table`では元となる列挙体を差し替える機能も利用できます。

### インポートオプション

![DataTableOptions_Original](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/a581d47c-94db-4a90-a514-933eb44b12f9)

エンジン標準のjsonやcsvからのインポートオプションではこのプラグインで追加される拡張データテーブルが選択できないため、同様の機能を持つインポートオプションを追加しています。  

![DataTableOptions_Idle](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/fa18a5b7-16ad-4f3c-a710-20b4fbde6c2c)
![DataTableOptions_Menu](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/9d17b54e-18e6-4015-b9bf-1dd1541e5666)
![DataTableOptions_Selected](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/0fd88a94-9468-4f02-9036-028b3443e7aa)

エンジン標準の物と違い、データテーブルで使う構造体や列挙体はApplyボタンを押した後に、新規でアセットを作る手順と同様に選択することができます。  
また、カーブテーブルなどエンジン標準のインポートオプションでのみ選択できますが、Cancelボタンを押すと続いてエンジン標準のインポートオプションが表示されるため、そちらをご利用ください。  

## サンプル

購入前に動作を確認できるようにサンプルプロジェクトのDevelopmentとShippingのパッケージがダウンロードできます。  
また、サンプルプロジェクトのエディタ上のデータを確認できるようにブループリントグラフやデータテーブルを確認できる画像とソースコードもダウンロードできます。

[サンプルをダウンロード](https://github.com/Naotsun19B/EnhancedDataTables-Document/releases/tag/Sample)

## ライセンス

[MITライセンス](https://ja.wikipedia.org/wiki/MIT_License)

## 作者

[Naotsun](https://twitter.com/Naotsun_UE)

## 履歴  

- (2024/08/03) v1.6  
  拡張データテーブルアセットのエディタを開いた状態で拡張データテーブルアセットの元となる構造体アセットに変更を加えるとクラッシュする不具合を修正しました  

- (2024/06/25) v1.5  
  プロジェクトランチャーからパッケージを作成する際に開発専用データが削除されない不具合を修正しました  

- (2024/04/24) v1.4  
  UE5.4に対応しました  
  スタンドアロンでプレイした際にクラッシュする不具合を修正しました  

- (2024/03/09) v1.3  
  再インポート時にもインポートオプションが表示される不具合を修正しました  
  複製ボタンで追加したデータがエディタを閉じると消える不具合を修正しました  
  UE5.1以前のバージョンでアセットのコンテキストメニューにエクスポートとソースファイルを開く項目が表示されない不具合を修正しました  
  アセットエディタからソースファイルを設定できない不具合を修正しました
  既に存在する行データの位置を変更してエディタを再起動すると変更がリセットされる不具合を修正しました  
  `Grouped Data Table`のアセットレジストリタグにグループIDの型名を追加しました  

- (2023/12/16) v1.2   
  開発専用の行を削除する必要があるプロジェクト設定からビルド構成を変更する機能を追加しました  
  行エディターでスクロールするときに開発専用のオーバーレイの位置がズレる問題を修正しました  

- (2023/11/07) v1.1   
  `Grouped Data Table`の内部情報が適切に更新されない不具合を修正しました  
  jsonやcsvから拡張データテーブルをインポートできる機能を追加しました  
  拡張データテーブルで再インポートが行えるように修正しました  

- (2023/11/01) v1.0   
  プラグインを公開  
