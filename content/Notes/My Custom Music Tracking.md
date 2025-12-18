While I'm [[Moving away from Spotify]], I realise I will miss my yearly Spotify Wrapped very dearly.

```kotlin
override fun onPlayingMetaChanged() {  
        for (listener in mMusicServiceEventListeners) {  
            listener.onPlayingMetaChanged()  
        }  
        lifecycleScope.launch(Dispatchers.IO) {  
            if (!PreferenceUtil.pauseHistory) {  
                repository.upsertSongInHistory(MusicPlayerRemote.currentSong)  
            }  
            val song = repository.findSongExistInPlayCount(MusicPlayerRemote.currentSong.id)  
                ?.apply { playCount += 1 }  
                ?: MusicPlayerRemote.currentSong.toPlayCount()  
  
            repository.upsertSongInPlayCount(song)  
  
//            val context = applicationContext()  
            logSong(this@AbsMusicServiceActivity, MusicPlayerRemote.currentSong)  
        }  
    }  
  
    fun logSong(context: Context, song: Song) {  
        try {  
            val filename = "log.txt"  
            val path = context.getExternalFilesDir(null)  
            val file = File(path, filename)  
            val formatter = SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ssZ")  
            val date = Date()  
            val now = formatter.format(date)  
            val data = "${song.id}\t${song.title}\t${song.artistName}\t${song.albumName}\t${song.trackNumber}\t${song.year}\t${song.duration}\t${now}\n"  
  
            file.appendText(data)  
            Log.d("LogSongPlay", "Written to log successfully")  
        } catch (e: Exception) {  
            // do nothing, don't care.  
        }  
    }
```