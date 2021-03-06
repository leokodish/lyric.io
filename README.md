[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]



<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://github.com/leokodish/mikashi">
    <img src="nodeserver/src/assets/Mikashi_Large.png" alt="Logo" width="300" height="300">
  </a>

  <h3 align="center">Mikashi</h3>

  <p align="center">
    Automated Lyric Search for Spotify
    <br />
    <a href="https://github.com/leokodish/mikashi"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://www.mikashi.net">View App</a>
    ·
    <a href="https://github.com/leokodish/mikashi/issues">Report Bug</a>
    ·
    <a href="https://github.com/leokodish/mikashi/issues">Request Feature</a>
  </p>
</p>



<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#privacy">Privacy</a></li>
    <li><a href="#why-did-i-get-a-404-song-not-found">Why Did I Get a 404 Song Not Found?</a></li>
    <li><a href="#architecture">Architecture</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

[![Product Name Screen Shot][product-screenshot]](https://www.mikashi.net)

Mikashi came about as a project centered around the Spotify Web API. I had been exploring uses for analyzing user data pertaining to liked songs and playlists, but it wasn’t until I saw that Spotify had created an API for the Spotify Player that I came up with this. This new API combined with my complete inability to hear song lyrics when I listened to songs on Spotify gave me the idea for a web app for automated lyric searching.

The name Mikashi comes from combining the kanji 見 ("see"), and the word 歌詞 ("lyrics"). 

Mikashi utilizes the Spotify Player API to get the user’s currently playing song, artist, and album. It then uses that data to construct a URL to fetch the HTML of the song’s page from either SongLyrics.com or Genius.com. Mikashi first tries to fetch the lyrics from SongLyrics and if it isn’t able to get them from there it tries Genius. 

Mikashi also allows users who don’t have Spotify accounts or users who just want to look up song lyrics directly to do that via its search feature. All the user has to do is enter the song and artist they are looking for and Mikashi fetches the lyrics for them.

### Built With
* [Node.js](https://nodejs.org/en/)
* [Express](https://expressjs.com/)
* [Pug](https://pugjs.org/api/getting-started.html)
* [Bootstrap](https://getbootstrap.com/)
* [Amazon Web Services](https://aws.amazon.com/)


<!-- USAGE EXAMPLES -->
## Usage
To use Mikashi, simply play music through Spotify (this can be on any of your devices, not just the one you have Mikashi open on), then navigate to [mikashi.net](https://www.mikashi.net) and click the button to log in with Spotify. You will be prompted to log in to your Spotify account to give Mikashi permission to access your music library. Once you have logged in, if your currently playing song is able to have its lyrics fetched, Mikashi will present the lyrics to you. When you want to change songs, all you have to do is press the "Update Song" button on the lyrics page and Mikashi will find the lyrics to your current song and update the page. 

**Mikashi Demo:**

![Mikashi Demo](nodeserver/src/assets/demo.gif)

<!-- Privacy -->
## Privacy
Mikashi does not store or save any user data. When you log in with Spotify, Mikashi requests an access token from Spotify that it can then use to make requests to the Spotify Web API. This access token limits Mikashi to only accessing data that has to do with the user’s music library and publicly available profile. 
Mikashi will not make any changes to your Spotify library. The access token is also deliberately set to expire after a set period of time, making Mikashi’s access to user data temporary and requiring the user to log in again to grant permission to access their music library.

**Authentication Flow:**
[![Spotify Authorization Flow][spotify-auth-diagram]](https://developer.spotify.com/documentation/general/guides/authorization-guide/)

*For more information: [Spotify Docs](https://developer.spotify.com/documentation/general/guides/authorization-guide/)*


<!-- Why Did I Get a 404 Song Not Found? -->
## Why Did I Get a 404 Song Not Found?
There are multiple reasons why a song may not be able to be found using Mikashi. In order to get song lyrics, the app needs to construct a URL to fetch the song lyrics from a  website like SongLyrics or Genius. For example, if the user were listening to “Memories” by Leonard Cohen and was trying to get the lyrics from Genius, then the app would need to construct the URL [https://genius.com/Leonard-cohen-memories-lyrics](https://genius.com/Leonard-cohen-memories-lyrics). If the constructed URL differed from this, then the URL would be incorrect and the response back from Genius would not contain the lyrics, resulting in Mikashi not being able to present lyrics back to the user. For most songs, the URL is able to be constructed easily since the URL just requires having the song name and artist, but there are several cases in which the data returned from the Spotify Web API makes it difficult or impossible to accurately construct the correct URL. 

One such case is when the user’s currently playing song features multiple artists. If the Spotify Web API returns an artist response object with multiple artists in it, I simply grab the first artist in the list and use that to construct the URL. The difficulty comes when Spotify doesn’t list all of the artists in the artist response object, but instead lists the featured artists directly in the song title (e.g. “Slippery (feat. Gucci Mane)”). Since I directly use the song name returned from Spotify to construct the URL without any string manipulation besides adding hyphens between whitespace, if there is extra text in the song name such as a features list or a note saying the song is a remastered version, it is likely that the constructed URL will be incorrect and the song will not be able to be fetched. (UPDATE: I have added some string manipulation to handle certain edge cases when it comes to song and artist names, but this is an iterative process as I continue to find songs that don't work for one reason or another)

Another case in which a song’s lyrics may not be able to be fetched is when the song or artist contains non-English characters. Multilingual URL addresses do exist and are valid web addresses, but SongLyrics and Genius only use English characters in the URLS they create for their pages. This is true even if the song or artist name is in a foreign script. For example, the song **あこがれ (Akogare)** by **大貫妙子 (Taeko Ohnuki)** is written using Japanese script, but Genius’ URL for the song will be [https://genius.com/Taeko-ohnuki-akogare-lyrics](https://genius.com/Taeko-ohnuki-akogare-lyrics), with the artist and song name written in Romanized form. The problem that this poses is that if the song or artist name returned from Spotify is written in a foreign script, there isn’t a reliable way to convert this data into a Romanized form that I can then use to construct a URL. There are services such as Google’s Translation API that can detect the language a word is in (which is necessary since the data coming in from Spotify could be in just about any language, since Spotify has music from all over the world), and provide a translation. However, a translation is not very helpful, since what I actually need in this case is a transliteration of the word, where the source word would be transferred into English in a way that maintained its pronunciation. Unfortunately, this means that for now Mikashi can only support foreign music as long as the song and artist data returned from Spotify is in English. 

Apologies in advance to the K-Pop fans, but unless the song’s title is in English or Romaja, it’s unlikely you'll be able to use Mikashi to find the lyrics to your favorite K-Pop songs. 


<!-- Architecture -->
## Architecture
Mikashi is written in Node.js and uses Express for its backend. Pug is used as a template engine to generate HTML for the front-end and uses CSS for front-end styling.

Mikashi was developed for the cloud and is hosted on AWS. It is built with a serverless architecture, so all of the infrastructure management and server provisioning is handled by AWS. AWS API Gateway creates and manages the REST API for Mikashi. When users go to [mikashi.net](https://www.mikashi.net) on their browser, the REST API is invoked and triggers the AWS Lambda function where a deployment package containing the code for the Mikashi app is stored. Lambda provisions a server (as needed) to run the app and API Gateway returns Mikashi’s entry point to the user for them to then use the app.  

The domain Mikashi.net is registered with Route 53. This combined with API Gateway allowed me to set a custom domain name for the app, so users can access the app through Mikashi.net rather than the verbose and unattractive URL that AWS auto-generates. 

**Mikashi Cloud Architecture**

[![Mikashi Cloud Architecture][aws-flow-diagram]](https://developer.spotify.com/documentation/general/guides/authorization-guide/) 


<!-- LICENSE -->
## License

Distributed under the MIT License. See [`LICENSE`](https://github.com/leokodish/mikashi/blob/master/LICENSE) for more information.


<!-- CONTACT -->
## Contact

Leo Kodish - [@website](https://leokodish.com) - mikashi.app@gmail.com

Project Link: [https://github.com/leokodish/mikashi](https://github.com/leokodish/mikashi)

<!-- MARKDOWN LINKS & IMAGES -->
[contributors-shield]: https://img.shields.io/github/contributors/leokodish/mikashi.svg?style=for-the-badge
[contributors-url]: https://github.com/leokodish/mikashi/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/leokodish/mikashi.svg?style=for-the-badge
[forks-url]: https://github.com/leokodish/mikashi/network/members
[stars-shield]: https://img.shields.io/github/stars/leokodish/mikashi?style=for-the-badge
[stars-url]: https://github.com/leokodish/mikashi/stargazers
[issues-shield]: https://img.shields.io/github/issues/leokodish/mikashi?style=for-the-badge
[issues-url]: https://github.com/leokodish/mikashi/issues
[license-shield]: https://img.shields.io/github/license/leokodish/mikashi.svg?style=for-the-badge
[license-url]: https://github.com/leokodish/mikashi/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/in/leokodish
[product-screenshot]: nodeserver/src/assets/Mikashi_Screenshot.png
[aws-flow-diagram]: nodeserver/src/assets/Mikashi_AWS_Flowchart.png
[spotify-auth-diagram]: nodeserver/src/assets/Spotify_Auth_Flow.png
