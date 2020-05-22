[Go back to the README](https://github.com/martendebruijn/meesterproef-1920)

# Genius API

While Nick was experimenting with the Spotify API I decided to experiment with the Genius API.

Firstly you have to register your application. You can do that [here](https://docs.genius.com/). Here you fill in the URL of your website and the redirect URI. Here you get a client ID and a secret key from Genius.

With a client ID, a redirect URI, a scope and a state you can provide the user with the link that is shown below. When the user goes to this link, he will be promed by an authentication screen. Here he can allow the application to use data from Genius.

```js
`https://api.genius.com/oauth/authorize?client_id=${client_id}&redirect_uri=${redirect_uri}&scope=${scope}&state=${state}&response_type=code`;
```

If the user accept he will be redirect to our redirect URI with a code and our state.

```js
`https://YOUR_REDIRECT_URI/?code=CODE&state=SOME_STATE_VALUE.`;
```

You can use this code to do a `POST` request to `https://api.genius.com/oauth/token` to get an access token back.

```js
const fetch = require('node-fetch');
const client_id = process.env.GENIUS_CLIENT_ID,
  secret = process.env.GENIUS_SECRET,
  redirect_uri = process.env.GENIUS_REDIRECT,
  scope = 'me',
  state = process.env.GENIUS_STATE;
var access_token;

const baseURL = 'https://api.genius.com/';
async function getAccesToken(code) {
  const extendedURL = 'oauth/token',
    bodyData = {
      code: code,
      client_id: client_id,
      client_secret: secret,
      redirect_uri: redirect_uri,
      response_type: 'code',
      grant_type: 'authorization_code',
    };
  const url = baseURL + extendedURL;
  const headers = {
    Accept: 'application/json',
    'Content-Type': 'application/json',
  };
  const response = await fetch(url, {
    method: 'POST',
    headers: headers,
    body: JSON.stringify(bodyData),
  });
  const jsonData = await response.json();
  access_token = jsonData.access_token;
}
```

When you have an access token you can do a `GET` request to collect data. I've made a simple form with an input field where we can search Genius from our application.

```js
async function search(query) {
  const extendedURL = 'search?q=';
  const url = baseURL + extendedURL + query;
  const headers = {
    Accept: 'application/json',
    'Content-Type': 'application/json',
    'User-Agent': 'CompuServe Classic/1.22',
    Host: 'api.genius.com',
    Authorization: `Bearer ${access_token}`,
  };
  const response = await fetch(url, {
    method: 'GET',
    headers: headers,
  });
  const jsonData = await response.json();
  const myObject = jsonData.response.hits;
  const result = myObject[0].result;
  const title = result.title;
  const cover = result.song_art_image_url;
  const artist = result.primary_artist.name;
  return { title, cover, artist };
}
```

#

[Go back to the README](https://github.com/martendebruijn/meesterproef-1920)
