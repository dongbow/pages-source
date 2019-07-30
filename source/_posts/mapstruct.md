---
title: mapstruct简介和使用
date: 2019-07-23 21:21:34
categories: mapstruct
tags:
	- mapstruct
---
## 简介
mapstrct是一个很好注释处理的框架，解决繁琐domain之间值的转换，节约开发的时间，同时相对应copyProperty的好处是没有使用反射技术，使性能更优。mapstrut一共两个主要的包，org.mapstruct.mapstruct包含里面常用的注释，org.mapstruct.mapstruct-processor处理注释的实现。
<!-- more -->

## 官方文档
[文档传送门](http://mapstruct.org/documentation/stable/reference/html/)

## API & 使用
### MAVEN
```xml
<!-- 指定版本 -->
<properties>
	<org.mapstruct.version>1.3.0.Final</org.mapstruct.version>
</properties>

<!-- 引入依赖 -->
<dependencies>
	<!--mapstruct主键-->
	<dependency>
		<groupId>org.mapstruct</groupId>
		<artifactId>mapstruct</artifactId>
		<version>${org.mapstruct.version}</version>
	</dependency>
	<!-- mapstruct插件 可以直接点击查看mapstruct的实现类 -->
	<dependency>
		<groupId>org.mapstruct</groupId>
		<artifactId>mapstruct-jdk8</artifactId>
		<version>${org.mapstruct.version}</version>
	<dependency>
	<!-- mapstruct实现类执行器 -->
	<dependency>
		<groupId>org.mapstruct</groupId>
		<artifactId>mapstruct-processor</artifactId>
		<version>${org.mapstruct.version}</version>
		<scope>private</scope>
	</dependency>
</dependencies>
```
### 基本使用
1. 定义一个接口，并使用`@Mapper` 
2. 定义一个转换的方法。方法上常用的注解  

```java
@Mapping(target="",source="")、
@Mappings({@Mapping(target="",source=""),@Mapping(target="",source="")})
```
**ps:如果target和source字段相同的则无需使用注解，mapstruct会自动为我们copy值的**  
方法名称上面如果有个可以忽略自动引入的注解,可以防止target copy source的时候报错
`@BeanMapping(ignoreByDefault=true)`

**eg:基本使用**
```java
/**
 * @author yinbingyu
 * @since 2019/07/30
 */
@Data
@NoArgsConstructor
@FieldDefaults(level = AccessLevel.PRIVATE)
@AllArgsConstructor
public class Item {
    Long itemId;
    String title;
}

@Data
@NoArgsConstructor
@FieldDefaults(level = AccessLevel.PRIVATE)
@AllArgsConstructor
public class Sku {
    Long skuId;
    String code;
    Integer price;
}

@Data
@NoArgsConstructor
@FieldDefaults(level = AccessLevel.PRIVATE)
@AllArgsConstructor
public class SkuDTO {
    Long skuId;
    String skuCode;
    Integer skuPrice;
    Long itemId;
    String itemName;
}

@Mapper
public interface ItemConvert {

    ItemConvert INSTANCE = Mappers.getMapper(ItemConvert.class);

    @BeanMapping(ignoreByDefault = true)
    @Mappings({
           @Mapping(source = "sku.code", target = "skuCode"),
           @Mapping(source = "sku.price", target = "skuPrice"),
           @Mapping(source = "item.title", target = "itemName")
    })
    SkuDTO domain2Dto(Sku sku, Item item);
}
```
mapstruct 自动实现接口代码如下

```java
@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2019-07-30T18:14:08+0800",
    comments = "version: 1.3.0.Final, compiler: javac, environment: Java 1.8.0_101 (Oracle Corporation)"
)
public class ItemConvertImpl implements ItemConvert {

    @Override
    public SkuDTO domain2Dto(Sku sku, Item item) {
        if ( sku == null && item == null ) {
            return null;
        }

        SkuDTO skuDTO = new SkuDTO();

        if ( sku != null ) {
            skuDTO.setSkuPrice( sku.getPrice() );
            skuDTO.setSkuCode( sku.getCode() );
        }
        if ( item != null ) {
            skuDTO.setItemName( item.getTitle() );
        }

        return skuDTO;
    }
}

```
因为使用了`@BeanMapping(ignoreByDefault = true)`，所以可以看到只有我们指定的那些字段才进行了赋值，如果，我们去掉@BeanMapping，他的实现类是这样的

```

@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2019-07-30T18:17:19+0800",
    comments = "version: 1.3.0.Final, compiler: javac, environment: Java 1.8.0_101 (Oracle Corporation)"
)
public class ItemConvertImpl implements ItemConvert {

    @Override
    public SkuDTO domain2Dto(Sku sku, Item item) {
        if ( sku == null && item == null ) {
            return null;
        }

        SkuDTO skuDTO = new SkuDTO();

        if ( sku != null ) {
            skuDTO.setSkuPrice( sku.getPrice() );
            skuDTO.setSkuCode( sku.getCode() );
            skuDTO.setSkuId( sku.getSkuId() );
        }
        if ( item != null ) {
            skuDTO.setItemName( item.getTitle() );
            skuDTO.setItemId( item.getItemId() );
        }

        return skuDTO;
    }
}
```
3. 类型的转换有三种特殊的  
- 常见基本类型的不同，`@Mapping` 会自动转换 
- bean不同的类型的转换 会创建或者调用其他的方法 
- collection类型，如果相同的类型他会重新创建一个新的实例集合，并且赋值，如果不同的则会分开进行赋值。

```java

/**
 * @author yinbingyu
 * @since 2019/07/30
 */
@Data
@NoArgsConstructor
@FieldDefaults(level = AccessLevel.PRIVATE)
@AllArgsConstructor
public class Item1 {
    Long itemId;
    String title;
}

/**
 * @author yinbingyu
 * @since 2019/07/30
 */
@Data
@NoArgsConstructor
@FieldDefaults(level = AccessLevel.PRIVATE)
@AllArgsConstructor
public class Item2 {
    Long itemId;
    String title;
}

@Data
@NoArgsConstructor
@FieldDefaults(level = AccessLevel.PRIVATE)
@AllArgsConstructor
public class Sku2 {
    Long skuId;
    String skuCode;
    String skuPrice;
    List<String> nameList;
    Item1 item;
}

@Data
@NoArgsConstructor
@FieldDefaults(level = AccessLevel.PRIVATE)
@AllArgsConstructor
public class SkuDTO2 {
    Long skuId;
    String skuCode;
    Long skuPrice;
    List<String> nameList;
    Item2 item;
}

@Mapper
public interface ItemConvert {

    ItemConvert INSTANCE = Mappers.getMapper(ItemConvert.class);

    SkuDTO2 domain2Dto(Sku2 sku2);
}

// 以下为mapstruct自动生成
public class ItemConvertImpl implements ItemConvert {

    @Override
    public SkuDTO2 domain2Dto(Sku2 sku2) {
        if ( sku2 == null ) {
            return null;
        }

        SkuDTO2 skuDTO2 = new SkuDTO2();

        skuDTO2.setSkuId( sku2.getSkuId() );
        skuDTO2.setSkuCode( sku2.getSkuCode() );
        if ( sku2.getSkuPrice() != null ) {
            skuDTO2.setSkuPrice( Long.parseLong( sku2.getSkuPrice() ) );
        }
        List<String> list = sku2.getNameList();
        if ( list != null ) {
            skuDTO2.setNameList( new ArrayList<String>( list ) );
        }
        skuDTO2.setItem( item1ToItem2( sku2.getItem() ) );

        return skuDTO2;
    }
    
   protected Item2 item1ToItem2(Item1 item1) {
    if ( item1 == null ) {
        return null;
    }

    Item2 item2 = new Item2();

    item2.setItemId( item1.getItemId() );
    item2.setTitle( item1.getTitle() );

    return item2;
}
```
4. mapstruct可以自定义类型转换，只要入参的类型和返回值类型需要的一致，就会自动调用。同时mapstruct的定义不仅仅可以使用接口，也可以使用抽象类。

```java

/**
 * @author yinbingyu
 * @since 2019/07/30
 */
@Mapper
public interface ItemConvert {

    ItemConvert INSTANCE = Mappers.getMapper(ItemConvert.class);

    SkuDTO2 domain2Dto(Sku2 sku2);

    default Item2 item1ToItem2(Item1 item1) {
        if (item1 == null) {
            return null;
        }

        Item2 item2 = new Item2();

        item2.setItemId(11111L);
        item2.setTitle(item1.getTitle());

        return item2;
    }
}

// mapstruct 的实现类
@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2019-07-30T18:38:11+0800",
    comments = "version: 1.3.0.Final, compiler: javac, environment: Java 1.8.0_101 (Oracle Corporation)"
)
public class ItemConvertImpl implements ItemConvert {

    @Override
    public SkuDTO2 domain2Dto(Sku2 sku2) {
        if ( sku2 == null ) {
            return null;
        }

        SkuDTO2 skuDTO2 = new SkuDTO2();

        skuDTO2.setSkuId( sku2.getSkuId() );
        skuDTO2.setSkuCode( sku2.getSkuCode() );
        if ( sku2.getSkuPrice() != null ) {
            skuDTO2.setSkuPrice( Long.parseLong( sku2.getSkuPrice() ) );
        }
        List<String> list = sku2.getNameList();
        if ( list != null ) {
            skuDTO2.setNameList( new ArrayList<String>( list ) );
        }
        // mapstruct直接调用的是在接口中自定义的实现
        skuDTO2.setItem( item1ToItem2( sku2.getItem() ) );

        return skuDTO2;
    }
}
```

5. mapstruct支持多个参数的，如果名字一样则会自动映射，不同则需要使用`@Mapping`，如果多个参数的都有`target`的属性名需要制定是哪个source的属性名，否则会报错

```java
/**
 * @author yinbingyu
 * @since 2019/07/01
 */
@Mapper
public interface CarAssertApiMapper {

    CarAssertApiMapper INSTANCE = Mappers.getMapper(CarAssertApiMapper.class);

    /**
     * 两个do转成一个dto
     *
     * @param purchaseBillItemDO 采购明细单
     * @param carManageDO        车型
     * @return 目的dto
     */
    @Mappings({
            @Mapping(target = "modelCode", source = "carManageDO.modelCode"),
            @Mapping(target = "modelName", source = "carManageDO.modelName"),
            @Mapping(target = "modelColor", source = "carManageDO.modelColor"),
            @Mapping(target = "modelColorCode", source = "carManageDO.modelColorCode"),
            @Mapping(target = "interiorColor", source = "carManageDO.interiorColor"),
            @Mapping(target = "interiorColorCode", source = "carManageDO.interiorColorCode"),
            @Mapping(target = "purchaseBillItemId", source = "purchaseBillItemDO.billItemId")
    })
    PurchaseCreateDTO doToDto(PurchaseBillItemDO purchaseBillItemDO, CarManageDO carManageDO);
}
```

mapstruct可以直接将一个基本类型的字段直接映射对应的bean字段 

```
/**
 * @author yinbingyu
 * @since 2019/05/29
 */
@Mapper
public interface CarMapper {

    CarMapper INSTANCE = Mappers.getMapper(CarMapper.class);

    /**
     * 多个参数转换为targetDTO
     *
     * @param carDO       source car DO
     * @param warehouseDO source warehouse DO
     * @param userName    操作人 这个会直接映射到目标bean的相同字段属性上，
     *                    若是属性名不一致，则需要使用@Mapping
     * @return target DTO
     * ps:多个mapping可能使用Mappings更方便
     * @Mappings({
     * @Mapping(source = "warehouseDO.plate_cities", target = "plateCities")
     * })
     */
    @Mapping(source = "warehouseDO.plate_cities", target = "plateCities")
    CarDTO severalSourceToDTo(CarDO carDO, WarehouseDO warehouseDO, String userName);
}
```

6. mapstruct不仅仅可以create instance, 也可以更新instance，使用`@MappingTarget`

```java
/**
 * @author yinbingyu
 * @since 2019/05/29
 */
@Mapper
public interface CarMapper {

    CarMapper INSTANCE = Mappers.getMapper(CarMapper.class);

  /**
     * 更新DTO里的值
     *
     * @param carDTO      target update instance
     * @param inventoryDO source do
     */
    void updateCarDTO(@MappingTarget CarDTO carDTO, InventoryDO inventoryDO);
}
```
7. 如果两个方向入参和返回值是相反，但是类型却是一样的，则可以使用反向映射`@InheritInverseConfiguration`,如果有多个复合条件的则可以指定固定的一个方法

```java
/**
 * @author yinbingyu
 * @since 2019/05/30
 */
@Mapper(typeConversionPolicy = ReportingPolicy.IGNORE)
public abstract class CustomerMapper {

    public final static CustomerMapper INSTANCE = Mappers.getMapper(CustomerMapper.class);

    /**
     * do转成dto
     *
     * @param customerDO
     * @return
     */
    @Mapping(source = "items", target = "memberItems")
    abstract CustomerDTO customerCovert(CustomerDO customerDO);

    /**
     * do 转dto
     *  作用和上面的转换是相同的
     * @param customerDO1
     * @return
     */
    @Mapping(source = "items", target = "memberItems")
    abstract CustomerDTO customerDTO(CustomerDO customerDO1);

    /**
     * dto convert do
     *
     * @param customerDTO
     * @return
     */
    @InheritInverseConfiguration(name = "customerCovert")
    abstract CustomerDO dtoCoverDTO(CustomerDTO customerDTO);
}
```
8. mapstruct的依赖注入方式实现

```java

/**
 * @author yinbingyu
 * @since 2019/06/03
 */
@Mapper(componentModel = "spring")
public interface CarCdiMapper {

    /**
     * 单个参数的直接使用
     *
     * @param carDO source do
     * @return target dto
     */
    CarDTO doToDTO(CarDO carDO);
}

/**
 * @author yinbingyu
 * @since 2019/05/29
 */
public class CarMapperTest {

    @Autowired
    private CarCdiMapper carCdiMapper;

    @Test
    public void testCDI() {
        CarDO carDO = CarDO.builder()
                .newCarGuidePrice(15880000L)
                .brandCode("brand-74")
                .brandName("JEEP")
                .seriesCode("series-50225")
                .seriesName("自由侠")
                .modelCode("14475-n")
                .modelName("2017款 自由侠 180T 自动劲能版")
                .modelColor("皓白")
                .modelColorCode("#FFFFFF")
                .build();
        CarDTO carDTO = carCdiMapper.doToDTO(carDO);
        System.out.println(carDTO);
    }
}

// 其他的请参考
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.CLASS)
public @interface Mapper {
    /**
     * Specifies the component model to which the generated mapper should
     * adhere. Supported values are
     * <ul>
     * <li> {@code default}: the mapper uses no component model, instances are
     * typically retrieved via {@link Mappers#getMapper(Class)}</li>
     * <li>
     * {@code cdi}: the generated mapper is an application-scoped CDI bean and
     * can be retrieved via {@code @Inject}</li>
     * <li>
     * {@code spring}: the generated mapper is a Spring bean and
     * can be retrieved via {@code @Autowired}</li>
     * <li>
     * {@code jsr330}: the generated mapper is annotated with {@code @javax.inject.Named} and
     * {@code @Singleton}, and can be retrieved via {@code @Inject}</li>
     * </ul>
     * The method overrides an unmappedTargetPolicy set in a central configuration set
     * by {@link #config() }
     *
     * @return The component model for the generated mapper.
     */
    String componentModel() default "default";
}
```
### Data type conversions
1. 可以自动类型转换的  
常见的类型及其他们的包装类mapstruct都是进行自动类型转换的。同时有些类型是可以转成string类型的。
mapstruct也是支持自定义类型的转换的`@Mapper(uses = 自定义转换类.class)`
当然也支持格式化，但是格式化的的参数同`java.text.DecimalFormat`

```java
/**
 * @author yinbingyu
 * @since 2019/06/04
 */
@Mapper(uses = BooleanStrategy.class)
public interface InventoryMapper {

    InventoryMapper INSTANCE = Mappers.getMapper(InventoryMapper.class);

    /**
     * DO 转 DTO
     *
     * @param inventoryDO
     * @return
     */
    @Mappings({
            @Mapping(target = "actualOutWarehouseDate", dateFormat = "yyyy-MM-dd HH:mm:ss"),
            @Mapping(target = "totalPrice", numberFormat = "\\u00A4#,###,###.00"),
            @Mapping(target = "inWaySubStatusEnum", source = "inventoryStatusEnum")
    })
    InventoryDTO doToDto(InventoryDO inventoryDO);

    /**
     * dto 转 DO
     * ignore可以忽略某个字段@BeanMapping可以忽略 所有满足自动转换的字段
     *
     * @param inventoryDTO
     * @return
     */
    @Mappings({
            @Mapping(target = "actualOutWarehouseDate", dateFormat = "yyyy-MM-dd HH:mm:ss"),
            @Mapping(target = "inventoryStatusEnum", source = "inWaySubStatusEnum"),
            @Mapping(target = "vin", ignore = true)
    })
    InventoryDO dtoToDo(InventoryDTO inventoryDTO);
}

/**
 * 自定义类型转换策略
 *
 * @author yinbingyu
 * @since 2019/06/04
 */
public class BooleanStrategy {

    public String booleanToString(Boolean value) {

        if (value == null) {
            return null;
        }

        return value ? "是" : "否";
    }

    public Integer booleanToInteger(Boolean value) {
        if (value == null) {
            return null;
        }

        return value ? 1 : 0;
    }

    public Boolean IntegerToBoolean(Integer value) {
        if (value == null) {
            return null;
        }

        return value == 0 ? false : true;
    }
}

// mapstuct自动实现

@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2019-07-15T14:03:23+0800",
    comments = "version: 1.3.0.Final, compiler: javac, environment: Java 1.8.0_101 (Oracle Corporation)"
)
public class InventoryMapperImpl implements InventoryMapper {

    private final BooleanStrategy booleanStrategy = new BooleanStrategy();

    @Override
    public InventoryDTO doToDto(InventoryDO inventoryDO) {
        if ( inventoryDO == null ) {
            return null;
        }

        InventoryDTOBuilder inventoryDTO = InventoryDTO.builder();

        if ( inventoryDO.getInventoryStatusEnum() != null ) {
            inventoryDTO.inWaySubStatusEnum( inventoryDO.getInventoryStatusEnum().name() );
        }
        inventoryDTO.vin( inventoryDO.getVin() );
        inventoryDTO.inventoryMainStatus( inventoryDO.getInventoryMainStatus() );
        inventoryDTO.inventorySubStatus( inventoryDO.getInventorySubStatus() );
        inventoryDTO.actualStorageDate( inventoryDO.getActualStorageDate() );
        if ( inventoryDO.getActualOutWarehouseDate() != null ) {
            inventoryDTO.actualOutWarehouseDate( new SimpleDateFormat( "yyyy-MM-dd HH:mm:ss" ).format( inventoryDO.getActualOutWarehouseDate() ) );
        }
        if ( inventoryDO.getTotalPrice() != null ) {
            inventoryDTO.totalPrice( createDecimalFormat( "\u00A4#,###,###.00" ).format( inventoryDO.getTotalPrice() ) );
        }
        inventoryDTO.canSale( booleanStrategy.booleanToString( inventoryDO.getCanSale() ) );
        inventoryDTO.canMove( booleanStrategy.booleanToInteger( inventoryDO.getCanMove() ) );

        return inventoryDTO.build();
    }

    @Override
    public InventoryDO dtoToDo(InventoryDTO inventoryDTO) {
        if ( inventoryDTO == null ) {
            return null;
        }

        InventoryDOBuilder inventoryDO = InventoryDO.builder();

        if ( inventoryDTO.getInWaySubStatusEnum() != null ) {
            inventoryDO.inventoryStatusEnum( Enum.valueOf( InventoryStatusEnum.class, inventoryDTO.getInWaySubStatusEnum() ) );
        }
        inventoryDO.inventoryMainStatus( inventoryDTO.getInventoryMainStatus() );
        inventoryDO.inventorySubStatus( inventoryDTO.getInventorySubStatus() );
        inventoryDO.actualStorageDate( inventoryDTO.getActualStorageDate() );
        try {
            if ( inventoryDTO.getActualOutWarehouseDate() != null ) {
                inventoryDO.actualOutWarehouseDate( new SimpleDateFormat( "yyyy-MM-dd HH:mm:ss" ).parse( inventoryDTO.getActualOutWarehouseDate() ) );
            }
        }
        catch ( ParseException e ) {
            throw new RuntimeException( e );
        }
        if ( inventoryDTO.getTotalPrice() != null ) {
            inventoryDO.totalPrice( new BigDecimal( inventoryDTO.getTotalPrice() ) );
        }
        if ( inventoryDTO.getCanSale() != null ) {
            inventoryDO.canSale( Boolean.parseBoolean( inventoryDTO.getCanSale() ) );
        }
        inventoryDO.canMove( booleanStrategy.IntegerToBoolean( inventoryDTO.getCanMove() ) );

        return inventoryDO.build();
    }

    private DecimalFormat createDecimalFormat( String numberFormat ) {

        DecimalFormat df = new DecimalFormat( numberFormat );
        df.setParseBigDecimal( true );
        return df;
    }
}
```
**ps:那些需要可能失去精度的转换可以控制他们的方案，是否需要抛异常默认是 `ReportingPolicy typeConversionPolicy() default ReportingPolicy.IGNORE`;**

2. 自定义类型转换  
- 一种是上面说到的`@Mapper(uses = BooleanStrategy.class)`
- 另外一种是需要自定义多个相同的映射法则的时候，可以使用`@Qualifier`注解

3. 默认值的问题  
defaultValue 必须是target和source必须都有的情况下才可以赋值默认值，只有target没有source的时候，可以使用constant

```java
@Mapper(uses = StringListMapper.class)
public interface SourceTargetMapper {

    SourceTargetMapper INSTANCE = Mappers.getMapper( SourceTargetMapper.class );

    @Mapping(target = "stringProperty", source = "stringProp", defaultValue = "undefined")
    @Mapping(target = "longProperty", source = "longProp", defaultValue = "-1")
    @Mapping(target = "stringConstant", constant = "Constant Value")
    @Mapping(target = "integerConstant", constant = "14")
    @Mapping(target = "longWrapperConstant", constant = "3001")
    @Mapping(target = "dateConstant", dateFormat = "dd-MM-yyyy", constant = "09-01-2014")
    @Mapping(target = "stringListConstants", constant = "jack-jill-tom")
    Target sourceToTarget(Source s);
}
```
官网说明
> If s.getStringProp() == null, then the target property stringProperty will be set to "undefined" instead of applying the value from s.getStringProp(). If s.getLongProperty() == null, then the target property longProperty will be set to -1. The String "Constant Value" is set as is to the target property stringConstant. The value "3001" is type-converted to the Long (wrapper) class of target property longWrapperConstant. Date properties also require a date format. The constant "jack-jill-tom" demonstrates how the hand-written class StringListMapper is invoked to map the dash-separated list into a List<String>.

### Mapping Collection
1. 原理是遍历source collection然后转换类型，put到target collection中
如果是可以自动转换的则自动转换，同date type conversion；若是无法自动转换的，则会查看是否有可以调用的类型

```java
@Mapper(imports = Date.class)
public interface CarMapper {

    CarMapper INSTANCE = Mappers.getMapper(CarMapper.class);
    
     /**
     * 基本类型collection的转换
     *
     * @param integers
     * @return
     */
    Set<String> integerSetToStringSet(Set<Integer> integers);

    /**
     * 调用存在已有方法的转换
     *
     * @param cars
     * @return
     */
    List<CarDTO> carsToCarDtos(List<CarDO> cars);

    /**
     * 单个参数的直接使用
     *
     * @param carDO source do
     * @return target dto
     */
    @Mapping(target = "userName", constant = "yby")
    @Mapping(target = "modelColor", source = "modelColor", defaultValue = "标致")
    @Mapping(target = "createTime", dateFormat = "yyyy-MM-dd HH:mm:ss", expression = "java(new Date())")
    CarDTO doToDTO(CarDO carDO);
}

// mapstruct实现

@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2019-07-30T19:27:05+0800",
    comments = "version: 1.3.0.Final, compiler: javac, environment: Java 1.8.0_101 (Oracle Corporation)"
)
public class CarMapperImpl implements CarMapper {
 
    @Override
    public Set<String> integerSetToStringSet(Set<Integer> integers) {
        if ( integers == null ) {
            return null;
        }

        Set<String> set = new HashSet<String>( Math.max( (int) ( integers.size() / .75f ) + 1, 16 ) );
        for ( Integer integer : integers ) {
            set.add( String.valueOf( integer ) );
        }

        return set;
    }

    @Override
    public List<CarDTO> carsToCarDtos(List<CarDO> cars) {
        if ( cars == null ) {
            return null;
        }

        List<CarDTO> list = new ArrayList<CarDTO>( cars.size() );
        for ( CarDO carDO : cars ) {
            list.add( doToDTO( carDO ) );
        }

        return list;
    }

    @Override
    public CarDTO doToDTO(CarDO carDO) {
        if ( carDO == null ) {
            return null;
        }

        CarDTO carDTO = new CarDTO();

        if ( carDO.getModelColor() != null ) {
            carDTO.setModelColor( carDO.getModelColor() );
        }
        else {
            carDTO.setModelColor( "标致" );
        }
        if ( carDO.getNewCarGuidePrice() != null ) {
            carDTO.setNewCarGuidePrice( BigDecimal.valueOf( carDO.getNewCarGuidePrice() ) );
        }
        carDTO.setBrandCode( carDO.getBrandCode() );
        carDTO.setBrandName( carDO.getBrandName() );
        carDTO.setSeriesCode( carDO.getSeriesCode() );
        carDTO.setSeriesName( carDO.getSeriesName() );
        carDTO.setModelCode( carDO.getModelCode() );
        carDTO.setModelName( carDO.getModelName() );
        carDTO.setModelColorCode( carDO.getModelColorCode() );

        carDTO.setUserName( "yby" );
        carDTO.setCreateTime( new Date() );

        return carDTO;
    }
}

```
没有可以调用的方法，则mapstruct尝试自己实现

```java

/**
 * @author yinbingyu
 * @since 2019/07/25
 */
@Mapper
public interface WarehouseMapper {

    WarehouseMapper INSTANCE = Mappers.getMapper(WarehouseMapper.class);

    /**
     * DTO转为VO
     *
     * @param warehouseValidDTOList
     * @return
     */
    List<WarehouseValidPageVO> dtoToVo(List<WarehouseValidDTO> warehouseValidDTOList);
}

// mapstruct 自己实现

@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2019-07-30T15:53:04+0800",
    comments = "version: 1.3.0.Final, compiler: javac, environment: Java 1.8.0_101 (Oracle Corporation)"
)
public class WarehouseMapperImpl implements WarehouseMapper {

    @Override
    public List<WarehouseValidPageVO> dtoToVo(List<WarehouseValidDTO> warehouseValidDTOList) {
        if ( warehouseValidDTOList == null ) {
            return null;
        }

        List<WarehouseValidPageVO> list = new ArrayList<WarehouseValidPageVO>( warehouseValidDTOList.size() );
        for ( WarehouseValidDTO warehouseValidDTO : warehouseValidDTOList ) {
            list.add( warehouseValidDTOToWarehouseValidPageVO( warehouseValidDTO ) );
        }

        return list;
    }

    protected WarehouseValidPageVO warehouseValidDTOToWarehouseValidPageVO(WarehouseValidDTO warehouseValidDTO) {
        if ( warehouseValidDTO == null ) {
            return null;
        }

        WarehouseValidPageVO warehouseValidPageVO = new WarehouseValidPageVO();

        warehouseValidPageVO.setId( warehouseValidDTO.getId() );
        warehouseValidPageVO.setWarehouseNo( warehouseValidDTO.getWarehouseNo() );
        warehouseValidPageVO.setWarehouseName( warehouseValidDTO.getWarehouseName() );
        warehouseValidPageVO.setPlateValid( warehouseValidDTO.getPlateValid() );

        return warehouseValidPageVO;
    }
}

```

2. map  

```java
public interface SourceTargetMapper {

    @MapMapping(valueDateFormat = "dd.MM.yyyy")
    Map<String, String> longDateMapToStringStringMap(Map<Long, Date> source);
}

// mapstruct自己实现
@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2019-07-30T19:35:22+0800",
    comments = "version: 1.3.0.Final, compiler: javac, environment: Java 1.8.0_101 (Oracle Corporation)"
)
public class SourceTargetMapperImpl implements SourceTargetMapper {

    @Override
    public Map<String, String> longDateMapToStringStringMap(Map<Long, Date> source) {
        if ( source == null ) {
            return null;
        }

        Map<String, String> map = new HashMap<String, String>( Math.max( (int) ( source.size() / .75f ) + 1, 16 ) );

        for ( java.util.Map.Entry<Long, Date> entry : source.entrySet() ) {
            String key = new DecimalFormat( "" ).format( entry.getKey() );
            String value = new SimpleDateFormat( "dd.MM.yyyy" ).format( entry.getValue() );
            map.put( key, value );
        }

        return map;
    }
}
```
3. stream类型转换  

```java
@Mapper(imports = Date.class)
public interface CarMapper {

    CarMapper INSTANCE = Mappers.getMapper(CarMapper.class);

    /**
     * 单个参数的直接使用
     *
     * @param carDO source do
     * @return target dto
     */
    @Mapping(target = "userName", constant = "yby")
    @Mapping(target = "modelColor", source = "modelColor", defaultValue = "标致")
    @Mapping(target = "createTime", dateFormat = "yyyy-MM-dd HH:mm:ss", expression = "java(new Date())")
    CarDTO doToDTO(CarDO carDO);

    /**
     * stream的类型转换
     *
     * @param integers
     * @return
     */
    Set<String> integerStreamToStringSet(Stream<Integer> integers);

    /**
     * stream的类型转换
     *
     * @param cars
     * @return
     */
    List<CarDTO> carsStreamToCarDtos(Stream<CarDO> cars);
}

// mapstruct 实现

@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2019-07-30T19:38:25+0800",
    comments = "version: 1.3.0.Final, compiler: javac, environment: Java 1.8.0_101 (Oracle Corporation)"
)
public class CarMapperImpl implements CarMapper {

    @Override
    public Set<String> integerStreamToStringSet(Stream<Integer> integers) {
        if ( integers == null ) {
            return null;
        }

        return integers.map( integer -> String.valueOf( integer ) )
        .collect( Collectors.toCollection( HashSet<String>::new ) );
    }

    @Override
    public List<CarDTO> carsStreamToCarDtos(Stream<CarDO> cars) {
        if ( cars == null ) {
            return null;
        }

        return cars.map( carDO -> doToDTO( carDO ) )
        .collect( Collectors.toCollection( ArrayList<CarDTO>::new ) );
    }
    
     @Override
    public CarDTO doToDTO(CarDO carDO) {
        .....
    }
}
```

### Expressions  
1. 格式 `expression = "java( 表达式 )"`

```
@Mapper
public interface SourceTargetMapper {

    SourceTargetMapper INSTANCE = Mappers.getMapper( SourceTargetMapper.class );

    @Mapping(target = "timeAndFormat",
         expression = "java( new org.sample.TimeAndFormat( s.getTime(), s.getFormat() ) )")
    Target sourceToTarget(Source s);
}
```
2. 其中使用的类名必须要指定到全路径，若是使用比较麻烦，也可以使用`imports`

```java
@Mapper(imports = {JSON.class, String.class, OptimusConfig.class})
public interface TicketOuterMapper {


    /**
     * 个人牌订单保险信息组装
     *
     * @param plateInfoDO        个人牌保险信息
     * @param insuranceCreateDTO 创建保险的原始目标信息
     * @return
     */
    @Mappings({
        
            @Mapping(target = "insuranceIdentityCardUrlList", expression = "java(JSON.parseArray(plateInfoDO.getBuyerIdPicList(), String.class))"),
            @Mapping(target = "financingSite", expression = "java(OptimusConfig.getValue(\"org_name\"))")
    })
    InsuranceCreateDTO personInfoDoToDto(@MappingTarget InsuranceCreateDTO insuranceCreateDTO, PersonalPlateInfoDO plateInfoDO);
}
```
3. Default Expressions  

```java
imports java.util.UUID;

@Mapper( imports = UUID.class )
public interface SourceTargetMapper {

    SourceTargetMapper INSTANCE = Mappers.getMapper( SourceTargetMapper.class );

    @Mapping(target="id", source="sourceId", defaultExpression = "java( UUID.randomUUID().toString() )")
    Target sourceToTarget(Source s);
}
```