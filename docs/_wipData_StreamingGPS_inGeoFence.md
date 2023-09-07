Event Hub Data Generator, user-defined payload example:

```
[
    {
        "latitude": "40.82018471253422",
        "longitude": "-102.97433999137931"
    }
]
```

```
SELECT latitude
    , longitude
    , ST_WITHIN(
            CreatePoint(latitude, longitude),
            CreatePolygon(
                CreatePoint(36.992426, -109.060253)
                , CreatePoint(36.992426, -102.041524)
                , CreatePoint(41.003444, -102.041524)
                , CreatePoint(41.003444, -109.060253)
                , CreatePoint(36.992426, -109.060253)
                )
        ) 
INTO rchaplerdls 
FROM rchaplereh
WHERE latitude IS NOT NULL and longitude IS NOT NULL
```

![image](https://github.com/richchapler/AzureSolutions/assets/44923999/705a6ed9-e672-4298-bb02-8fe7a56317a0)
