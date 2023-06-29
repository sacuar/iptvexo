# iptvexo

Here is the explanation of making an IPTV Play
```
```
link for m3u list:
```
https://iptv-org.github.io/iptv/languages/deu.m3u
```

Add these libraries in gradle.build dependency

```
    implementation "androidx.media3:media3-exoplayer:1.0.2"
    implementation "androidx.media3:media3-exoplayer-dash:1.0.2"
    implementation "androidx.media3:media3-ui:1.0.2"
    implementation 'com.github.bumptech.glide:glide:4.15.1'
    implementation "androidx.media3:media3-datasource-okhttp:1.0.2"
    implementation "androidx.media3:media3-session:1.0.2"
    implementation "androidx.media3:media3-exoplayer-hls:1.0.2"
```

The functions explained :

parsePlaylist
```
   private fun parsePlaylist(playlistData: String) {
            val lines = playlistData.split("\n")

            // Regex patterns to extract channel information
            val namePattern = """#EXTINF:-1.*,(.*)""".toRegex()
            val logoPattern = """tvg-logo="(.*?)"""".toRegex()
            val urlPattern = """^(https?://.*?)(?:$|\s)""".toRegex()

            channelList.clear()

            for (line in lines) {
                if (line.startsWith("#EXTINF:-1")) {
                    val nameMatchResult = namePattern.find(line)
                    val logoMatchResult = logoPattern.find(line)
                    val urlMatchResult = urlPattern.find(lines[lines.indexOf(line) + 1])

                    if (nameMatchResult != null && logoMatchResult != null && urlMatchResult != null) {
                        val name = nameMatchResult.groupValues[1]
                        val logoUrl = logoMatchResult.groupValues[1]
                        val videoUrl = urlMatchResult.groupValues[1]

                        channelList.add(Channel(name, logoUrl, videoUrl))
                    }
                }
            }

            channelListAdapter.notifyDataSetChanged()
        }

```
fetchChannelList
```
private fun fetchChannelList(playlistUrl: String) {
            val client = OkHttpClient()

            val request = Request.Builder()
                .url(playlistUrl)
                .build()

            client.newCall(request).enqueue(object : Callback {
                override fun onFailure(call: Call, e: IOException) {
                    // Handle failure
                    e.printStackTrace()
                }

                override fun onResponse(call: Call, response: Response) {
                    val playlist = response.body?.string()
                    val parsedChannels = parsePlaylist(playlist)

                    runOnUiThread {
                        channels.clear()
                        channels.addAll(parsedChannels)
                        displayChannels()
                        playChannel(currentChannelIndex)
                    }
                }
            })
        }
```
displayChannels
```
     private fun displayChannels() {
            logoContainer.removeAllViews()

            channels.forEachIndexed { index, channel ->
                val logoView = createLogoView()
                logoContainer.addView(logoView)

                loadLogo(logoView, channel.logoUrl)

                logoView.setOnClickListener {
                    playChannel(index)
                }
            }
        }
```
createLogoView
```
private fun createLogoView(): ImageView {
        val logoView = ImageView(this)
        val layoutParams = LinearLayout.LayoutParams(
            resources.getDimensionPixelSize(R.dimen.logo_size),
            resources.getDimensionPixelSize(R.dimen.logo_size)
        )
        val margin = resources.getDimensionPixelSize(R.dimen.logo_margin)
        layoutParams.setMargins(margin, margin, margin, margin)
        logoView.layoutParams = layoutParams
        return logoView
    }
```
loadLogo
```
   private fun loadLogo(imageView: ImageView, logoUrl: String) {
            Glide.with(this)
                .load(logoUrl)
                .into(imageView)
        }
```
playChannel
```
       private fun playChannel(channelIndex: Int) {
            if (channelIndex in channels.indices) {
                currentChannelIndex = channelIndex
                val channel = channels[channelIndex]
                val mediaItem = MediaItem.fromUri(Uri.parse(channel.channelUrl))
                exoPlayer.setMediaItem(mediaItem)
                exoPlayer.prepare()
                exoPlayer.play()
            }
        }
```
variables and building exoplayer and initiating view
```
 private lateinit var exoPlayer: ExoPlayer
        private lateinit var playerView: PlayerView
        private lateinit var logoContainer: LinearLayout

        private val channels = mutableListOf<Channel>()
        private var currentChannelIndex = 0

        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_main)

            playerView = findViewById(R.id.player_view)
            logoContainer = findViewById(R.id.logo_container)

            exoPlayer = ExoPlayer.Builder(this).build()
            playerView.player = exoPlayer

            fetchChannelList("https://iptv-org.github.io/iptv/languages/ben.m3u")
        }
```
class Channel
```
 data class Channel(val name: String, val channelUrl: String, val logoUrl: String)
```
dimension
```
<resources>
    <dimen name="logo_size">48dp</dimen>
    <dimen name="logo_margin">18dp</dimen>
</resources>
```

Manifest use internet permission , otherwise channels will not connect to internet and will not play
```
<uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```

