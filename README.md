# ExcelMapper

[![NuGet version](https://badge.fury.io/nu/ExcelMapper.svg)](http://badge.fury.io/nu/ExcelMapper)
[![Build status](https://ci.appveyor.com/api/projects/status/tyyg8905i24qv9pg/branch/master?svg=true)](https://ci.appveyor.com/project/mganss/excelmapper/branch/master)
[![codecov.io](https://codecov.io/github/mganss/ExcelMapper/coverage.svg?branch=master)](https://codecov.io/github/mganss/ExcelMapper?branch=master)

A library to map [POCO](https://en.wikipedia.org/wiki/Plain_Old_CLR_Object) objects to Excel files.

## Features

* Read and write Excel files
* Uses the pure managed [NPOI](https://github.com/tonyqus/npoi) library instead of the [Jet](https://en.wikipedia.org/wiki/Microsoft_Jet_Database_Engine) database engine for Excel access, thus enabling use in AnyCPU configurations
* Map to Excel files using header rows (column names) or column indexes (no header row)
* Optionally skip blank lines when reading
* Preserve formatting when saving back files
* Optionally let the mapper track objects
* Map columns to properties through convention, attributes or method calls
* Use custom or builtin data formats for numeric and DateTime columns

## Read objects from an Excel file

```C#
var products = new ExcelMapper("products.xlsx").Fetch<Product>();
```

This expects the Excel file to contain a header row with the column names. Objects are read from the first worksheet. If the column names equal the property names (ignoring case) no other configuration is necessary. The format of the Excel file (xlsx or xls) is autodetected.

## Map to specific column names

```C#
public class Product
{
  public string Name { get; set; }
  [Column("Number")]
  public int NumberInStock { get; set; }
  public decimal Price { get; set; }
}
```

This maps the column named `Number` to the `NumberInStock` property.

## Map to column indexes

```C#
public class Product
{
    [Column(1)]
    public string Name { get; set; }
    [Column(3)]
    public int NumberInStock { get; set; }
    [Column(4)]
    public decimal Price { get; set; }
}

var products = new ExcelMapper("products.xlsx") { HeaderRow = false }.Fetch<Product>();
```

Note that column indexes don't need to be consecutive. When mapping to column indexes, every property needs to be explicitly mapped through the `ColumnAttribute` attribute or the `AddMapping()` method.

## Map through method calls

```C#
var excel = new ExcelMapper("products.xls");
excel.AddMapping<Product>("Number", p => p.NumberInStock);
excel.AddMapping<Product>(1, p => p.NumberInStock);
excel.AddMapping(typeof(Product), "Number", "NumberInStock");
excel.AddMapping(typeof(Product), 1, "NumberInStock");
```

## Save objects

```C#
var products = new List<Product>
{
    new Product { Name = "Nudossi", NumberInStock = 60, Price = 1.99m },
    new Product { Name = "Halloren", NumberInStock = 33, Price = 2.99m },
    new Product { Name = "Filinchen", NumberInStock = 100, Price = 0.99m },
};

new ExcelMapper().Save("products.xlsx", products, "Products");
```

This saves to the worksheet named "Products". If you save objects after having previously read from an Excel file using the same instance of `ExcelMapper` the style of the workbook is preserved allowing use cases where an Excel template is filled with computed data.

## Track objects

```C#
var products = new ExcelMapper("products.xlsx").Fetch<Product>().ToList();
products[1].Price += 1.0m;
excel.Save("products.out.xlsx");
```

## Ignore properties

```C#
public class Product
{
    public string Name { get; set; }
    [Ignore]
    public int Number { get; set; }
    public decimal Price { get; set; }
}

// or

var excel = new ExcelMapper("products.xlsx");
excel.Ignore<Product>(p => p.Price);
```

## Use specific data formats

```C#
public class Product
{
    [DataFormat(0xf)]
    public DateTime Date { get; set; }

    [DataFormat("0%")]
    public decimal Number { get; set; }
}
```

You can use both [builtin formats](https://poi.apache.org/apidocs/org/apache/poi/ss/usermodel/BuiltinFormats.html) and [custom formats](https://support.office.com/en-nz/article/Create-or-delete-a-custom-number-format-78f2a361-936b-4c03-8772-09fab54be7f4). The default format for DateTime cells is 0x16 ("m/d/yy h:mm").
