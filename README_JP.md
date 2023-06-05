# EnhancedDataTables

<!--ts-->
* [概要](#概要)
* [動作環境](#動作環境)
* [機能と使い方](#機能と使い方)
  * [Development Data Table](#development-data-table)
    * [アセット](#アセット)
    * [エディタ](#エディタ)
    * [ノード](#ノード)
  * [Enum Based Data Table](#enum-based-data-table)
    * [アセット](#アセット)
    * [エディタ](#エディタ) 
    * [ノード](#ノード)

## 概要

このプラグインは複数の機能が拡張されたデータテーブルアセットを追加します。

## 動作環境

対象バージョン : 未定    
対象プラットフォーム : Windows, Mac, Linux (ランタイムモジュールはプラットフォームの制限無し)

## 機能と使い方

### Development Data Table

`Development Data Table`はクックビルドとテストビルドの際に削除される行データを追加することができる拡張データテーブルアセットです。  


#### アセット

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

![RowUserDefinedStruct](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/714980de-96c6-42c3-8427-3c01547d6a87)


#### エディタ

![DevelopmentDataTableEditor](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/80f4c736-7b8b-4cce-8614-e08ec8682071)

アセットエディタも拡張されていて、開発専用フラグが`true`の行は背景が開発専用を示す模様に変わります。  
開発専用の行かどうかは他の行のプロパティと同様に、エディタ上の詳細パネルから変更できます。  


#### ノード

![IsDevelopmentOnly](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/7a8034e9-edfb-4b44-9fcf-38434d0b40fe)

`Development Data Table`の指定した行が開発専用の行かどうかを取得することができます。


### Enum Based Data Table

`Enum Based Data Table`は行の構造体とは別に行名や順序の元となる列挙体を指定する列挙体をベースとした拡張データテーブルアセットです。  


#### アセット

![CreateEnumBasedDataTable](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/813edae0-d365-4bde-9f2c-1c1e74259b91)

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


#### エディタ

![EnumBasedDataTableEditor](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/fca04d30-c638-4223-8e81-f17d8b0d2bda)

アセットエディタも拡張されていて、`Development Data Table`と同様で開発専用フラグが`true`の行は背景が開発専用を示す模様に変わります。  
また、行の追加や削除、移動などは元となる列挙体をベースとするためエディタ上から操作することはできません。  
開発専用フラグは元となる列挙体がC++で定義されている場合はエディタ上から変更ません。  
元となる列挙体がアセットの場合は`Development Data Table`のアセットエディタと同様に詳細パネルから変更可能です。  


#### ノード

![IsValidRowName](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/74dc20f2-1618-4cdf-9cbf-3446a408ce98)

指定した行名が`Enum Based Data Table`の元となる列挙体に含まれているかどうかを取得することができます。  

![HasRowEnumDevelopmentOnly](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/374240da-ecde-4246-9c34-b36e19d946c4)

指定した行名が`Enum Based Data Table`の元となる列挙体で`DevelopmentOnly`のメタ指定子を持っているかどうかを取得することができます。  

![GetEnumBasedDataTableRowFromEnum](https://github.com/Naotsun19B/EnhancedDataTables-Document/assets/51815450/4283cc24-87c4-46fa-803d-1dbe9897e80c)

行の列挙体を指定して`Enum Based Data Table`から行のデータを取得することができます。  
`Enum Based Data Table`ピンに直接アセットを指定すると`Row Enum`ピンと`Out Row`ピンの型が紐づいたもとに変わります。  