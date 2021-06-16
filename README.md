# Role unison

[Docs]: http://www.cis.upenn.edu/~bcpierce/unison/download/releases/stable/unison-manual.html

## Proměnné

Zapnutí služby synchronizace vyžaduje vytvoření pole slovníků `unison_twopoint_synclist`.
Slovník v poli definuje synchronizaci mezi dvěma body ve filesystému. Ve většině případů
budeme chtít nastavit proměnnou `remote` specifikující, že synchronizace probíhá po SSH
mezi stroji A a B. Proměnná `unison_twopoint_synclist` musí být definována v `host_vars`
stroje, na kterém poběží replikační služba. Na stroji B je vytvořen adresář `backupdir`
pomocí funkce `delegate_to`.

Slovník `unison_twopoint_synclist[X]` má definované proměnné:

| Proměnná  | Povinná | Výchozí    | Popis |
| --------- | ------- | ---------- | ----- |
| name      | ano     |            | Jméno služby a konfiguračního souboru. |
| remote    | pokud se synchronizuje s jiným serverem  | | Doménové jméno/IP adresa vzdáleného serveru, se kterým bude probíhat synchronizace.
| root1     | ano     |            | První adresář synchronizace. |
| root2     | ano     |            | Druhý adresář synchronizace. |
| backupdir | ano     |            | Odkladiště na soubory, které `unison` vyřadí v případě konfliktu. |
| backup_retain | ne  | 90         | Soubory starší než `backup_retain` dní budou automaticky mazány z `backupdir` |
| interval  | ne      | 60         | Jak často se spouští synchronizace. |
| logfile   | ne      | omit       | Cesta k log souboru. Výchozí log viz [dokumentace][Docs] |
| ignore_paths | ne   |            | Seznam ignorovaných cest k souborům. Každá položká má následující paramtery |
| .style    | ne      | Path       | Typ ignorované vzoru (*Pattern style* dle dokumentace Unison |
| .pattern  | ano     |            | Vzor ignorované cesty |

V případě SSH cesty je třeba specifikovat uživatele, protože služba je spouštěna pod `root`.

## Příklady použití

Chceme oboustranně synchronizovat adresář `/srv/www/sites` mezi servery
`web1.logicworks` a `web2.logicworks`.

Příklad konfigurace na `web1.logicworks`, kde poběží unison služba:

```yaml
failover_mirror: 'web2.logicworks'
unison_twopoint_synclist:
  - name: 'websitessync'
    service: true
    remote: '{{ failover_mirror }}'
    root1: '/srv/www/sites'
    root2: 'ssh://{{ unison.user }}@{{ failover_mirror }}//srv/www/sites'
    backupdir: '/srv/www/unison-backup'
    interval: 60
    logfile: '{{ unison.logdir }}/websitessync.log'
```

Pro `web2.logicworks` stačí mít v konfiguraci nastavenu proměnnou`failover_mirror`:

```yaml
failover_mirror: web1.logicworks
```
