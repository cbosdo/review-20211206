<!-- .slide: data-state="divider" data-timing="20s" -->
# Pillar in Database Implementation 
## Quick recap


<!-- .slide: data-state="normal" -->
# Table Schema 
```sql
CREATE TABLE suseSaltPillar
(
    id NUMERIC NOT NULL,
    server_id NUMERIC,
    group_id NUMERIC,
    org_id NUMERIC,
    category VARCHAR NOT NULL,
    pillar JSONB NOT NULL,
    UNIQUE (server_id, category),
    UNIQUE (group_id, category),
    UNIQUE (org_id, category),
);
```


<!-- .slide: data-state="normal" -->
# Postgresql JSONB

- No need to serialize / parse
- **Dict items order not guaranteed!**

```kotlin
@TypeDefs({
        @TypeDef(name = "json", typeClass = JsonType.class)
})
@Entity
@Table(name = "suseSaltPillar")
public class Pillar implements Identifiable {

    @Type(type = "json")
    @Column(columnDefinition = "jsonb")
    private Map<String, Object> pillar = new TreeMap<>();
}
```


<!-- .slide: data-state="normal" -->
# Pillars in Java 

Global
```kotlin
Pillar.getGlobalPillars();
Pillar.createGlobalPillar(category, pillarData);
```

`MinionServer` or `ServerGroup`
```kotlin
obj.getPillarByCategory(category);
obj.setPillars(pillars);
obj.getPillars();
```


<!-- .slide: data-state="normal" -->
# Formulas in Database

- Category: `"formula-" + formulaName`
- Groups & Server Formulas
- `pillar["formula_order"]`


<!-- .slide: data-state="normal" -->
# How the minion gets it 

[`spacewalk/setup/salt/susemanager.conf`](http://github.com/uyuni-project/uyuni/tree/master/spacewalk/setup/salt/susemanager.conf)
```yaml
ext_pillar:
    - suma_minion: True
```

[`susemanager-utils/susemanager-sls/modules/pillar/suma_minion.py`](https://github.com/uyuni-project/uyuni/tree/master/susemanager-utils/susemanager-sls/modules/pillar/suma_minion.py)

```python
def ext_pillar(minion_id, pillar, *args):
    # Entry point
```

Getting pillar, formulas data and images data

Note:
- Why dropping the postgresql external pillar:
  - No way to know what formula holds what pillar data!
  - Add default values using metadata


<!-- .slide: data-state="normal" -->
# Migration 

- When changing formula data
- Two code paths!
