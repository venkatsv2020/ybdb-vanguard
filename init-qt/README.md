# ybdb-vanguard-enablement

## ybdb query tunings

### explain plan
```
\set ea 'explain(analyze, dist, verbose, costs, buffers, timing, summary)'
```

### count pushdown
``` sql
:ea select count(1) from track;
```

### distinct pushdown
```sql
:ea select distinct albumid from track where albumid>0;
```

### hash index
```sql
:ea select * from track where albumid = 0;
```

### range index
```
:ea select * from track where albumid >0;

create index track_albumid on track(albumid asc);

:ea select * from track where albumid >0;
```

### expression pushdown
```sql
:ea select * from track where upper(name) like 'THE TROOPER';
```

### index scan and index only scan
```sql
:ea select * from track where albumid=1;

:ea select trackid, composer from track where albumid=1;

create index track_albumid_1 on track(albumid) include(trackid, composer);
```

### partial index
```
create index employee_city on employee(city) where city not in ('Lethbridge', 'Edmonton');

:ea select * from employee where city='Lethbridge';

:ea select * from employee where city='Calgary';
```

### index forward and backward scan
```sql
create index customer_repid on customer(supportrepid asc);

:ea select * from customer order by supportrepid;

:ea select * from customer order by supportrepid desc;
```

### joins
```sql
SELECT t.TrackId, t.Name AS track_name, a.Title AS album_title, ar.Name AS artist_name
FROM Track t
INNER JOIN Album a ON t.AlbumId = a.AlbumId
INNER JOIN Artist ar ON a.ArtistId = ar.ArtistId
WHERE t.TrackId = 5;

:ea SELECT t.TrackId, t.Name AS track_name, a.Title AS album_title, ar.Name AS artist_name
FROM Track t
INNER JOIN Album a ON t.AlbumId = a.AlbumId
INNER JOIN Artist ar ON a.ArtistId = ar.ArtistId
WHERE t.TrackId = 5;
```

### hints
```sql
:ea /*+Leading ( ( ( a t ) ar ) ) */ SELECT t.TrackId, t.Name AS track_name, a.Title AS album_title, ar.Name AS artist_name
FROM Track t
INNER JOIN Album a ON t.AlbumId = a.AlbumId
INNER JOIN Artist ar ON a.ArtistId = ar.ArtistId
WHERE t.TrackId = 5;

:ea /*+Leading ( ( ar ( a t ) ) ) */ SELECT t.TrackId, t.Name AS track_name, a.Title AS album_title, ar.Name AS artist_name
FROM Track t
INNER JOIN Album a ON t.AlbumId = a.AlbumId
INNER JOIN Artist ar ON a.ArtistId = ar.ArtistId
WHERE t.TrackId = 5;

:ea /*+Leading ( ( ar ( a t ) ) ) NestLoop(a t) */ SELECT t.TrackId, t.Name AS track_name, a.Title AS album_title, ar.Name AS artist_name
FROM Track t
INNER JOIN Album a ON t.AlbumId = a.AlbumId
INNER JOIN Artist ar ON a.ArtistId = ar.ArtistId
WHERE t.TrackId = 5;
```


### batch nested loop
```sql
set yb_bnl_batch_size=256;

:ea SELECT p.PlaylistId, p.Name AS playlist_name, t.Name AS track_name, ar.Name AS artist_name
FROM Playlist p
INNER JOIN PlaylistTrack pt ON p.PlaylistId = pt.PlaylistId
INNER JOIN Track t ON pt.TrackId = t.TrackId
INNER JOIN Album a ON t.AlbumId = a.AlbumId
INNER JOIN Artist ar ON a.ArtistId = ar.ArtistId
WHERE p.PlaylistId = 3;

```

