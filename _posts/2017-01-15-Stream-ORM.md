
```plain
mvn archetype:generate -DgroupId=org.redlich.publications
  -DartifactId=publications -DarchetypeArtifactId=speedment-archetype-mysql
  -DarchetypeGroupId=com.speedment.archetypes -DinteractiveMode=false
  -DarchetypeVersion=3.0.1 && cd publications && mvn speedment:tool
```

[](https://github.com/speedment/speedment)

```plain
警告: Loading FXML document with JavaFX API of version 8.0.65 by JavaFX runtime of version 8.0.40
```

UI是JAVAFX 适合先数据库建模然后再生成数据库操作代码




```java
LowpriceApplication app = new LowpriceApplicationBuilder().withPassword("321").build();
TLowPriceManager tLowPriceManager = app.getOrThrow(TLowPriceManager.class);


TLowPrice price = new TLowPriceImpl();
price.setSrcCode("PEK");
price.setDstCode("SHA");

tLowPriceManager.persist(price);


Optional<TLowPrice> lowPrice = tLowPriceManager.stream().findAny();
System.out.println(lowPrice);

tLowPriceManager.stream().filter(TLowPrice.SRC_CODE.equal("PEK"))
        .map(TLowPrice.DST_CODE.setTo("PVG"))
        .forEach(tLowPriceManager.updater());
```

```plain
2017-01-16T03:20:44.941Z ERROR [main] (c.s.r.c.i.s.b.AbstractStreamBuilder) - The table lowprice.test.lowprice.t_low_price does not have any primary keys. Some operations like update() and remove() requires at least one primary key.
com.speedment.runtime.core.exception.SpeedmentException: The table lowprice.test.lowprice.t_low_price does not have any primary keys. Some operations like update() and remove() requires at least one primary key.
```

```plain
ALTER TABLE `lowprice`.`t_low_price`
ADD COLUMN `id` VARCHAR(45) NOT NULL DEFAULT 'AUTO_INCREMENT' AFTER `aircraft_type`,
ADD PRIMARY KEY (`id`);

Executing:
ALTER TABLE `lowprice`.`t_low_price`
ADD COLUMN `id` VARCHAR(45) NOT NULL DEFAULT 'AUTO_INCREMENT' AFTER `aircraft_type`,
ADD PRIMARY KEY (`id`);

Operation failed: There was an error while applying the SQL script to the database.
ERROR 1062: Duplicate entry 'AUTO_INCREMENT' for key 'PRIMARY'
SQL Statement:
ALTER TABLE `lowprice`.`t_low_price`
ADD COLUMN `id` VARCHAR(45) NOT NULL DEFAULT 'AUTO_INCREMENT' AFTER `aircraft_type`,
ADD PRIMARY KEY (`id`)


```

[JOOQ]()
