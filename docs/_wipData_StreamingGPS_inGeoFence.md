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
SELECT input.latitude
    , input.longitude
    , ST_WITHIN(
            CreatePoint(input.latitude, input.longitude),
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
```
