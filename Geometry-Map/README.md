# [Geometry Map](https://bl.ocks.org/owen8877/3c93a30c66b284df01972a902390fce7)
_By owen8877_

## How To Get The Data
From [admin 0-detail](http://www.naturalearthdata.com/downloads/110m-cultural-vectors/)
using `ogrinfo _FILE_ -sql '_FILE'` to see the abbr of our interests.
```bash
ogr2ogr \                                                                                      
  -f GeoJSON \
  -where "adm0_a3 IN ('CH1', 'TWN')" \
  subunits.json \
  ne_110m_admin_0_sovereignty.shp
```

## Reference
[Let's make a map](https://bost.ocks.org/mike/map/)