<template>
  <div class="main_wrapper">
    <div class="inner_wrapper">
      <h1>Playlist Generator!</h1>

      <div class="logged" v-if="logedIn">
        <p>Hello {{ user_name }}!</p>
        <div class="import" v-if="generated">
          <p>Import to your acount</p>
          <button v-on:click="createPlaylist">Import!</button>
        </div>
        <div v-if="tracks" class="tracks">
          <ul>
            <li v-for="(item, index) in tracks" :key="item.id">
              <a target="_blank" :href="item.external_urls.spotify">
                <p class="song">{{ item.name }} /  {{ item.artists[0].name}}</p>
              </a>
            </li>
          </ul>
        </div>

        <div class="generate">
          <p>Generate a Playlist↓</p>
          <button v-on:click="getRecommend">GENERATE!</button>
        </div>
        <a class="logout" href="/">Logout</a>
      </div>

      <div class="login" v-else>
        <p>Login to your Spotify acount!</p>
        <a class="btn" href="/api/login">
          Login
        </a>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      access_token: '',
      refresh_token: '',
      error: '',
      logedIn: false,
      user_name: '',
      user_id: '',
      tracks: {},
      track_uris: [],
      generated: false
    }
  },
  mounted() {
      var params = {};
      var e, r = /([^&;=]+)=?([^&;]*)/g;
      var q = window.location.hash.substring(1);
      while ( e = r.exec(q)) {
          params[e[1]] = decodeURIComponent(e[2]);
      }
      this.access_token = params.access_token;
      this.refresh_token = params.refresh_token;
      this.error = params.error;
      this.loggedin();
  },
  methods: {
    async loggedin() {
      if(this.error) {
        alert('There was an error during the authentication');
      }
      else if (this.access_token) {
        const res = await this.$axios.$get(
          'https://api.spotify.com/v1/me',
          {
            headers: {
              'Authorization': 'Bearer ' + this.access_token
            }
          }
        )
        this.logedIn = true
        this.user_name = res.display_name
        this.user_id = res.id
      }
      else {
        this.logedIn = false
      }
    },
    async createPlaylist() {
      const res = await this.$axios.$post(
        `https://api.spotify.com/v1/users/${this.user_id}/playlists`,
        {
          name: 'yoyoPlaylist'
        },
        {
          headers: {
            'Authorization': 'Bearer ' + this.access_token,
            'Content-Type': 'application/json'
          },
        }
      )
      var playlist_id = res.id

      const res_add = await this.$axios.$post(
        `https://api.spotify.com/v1/playlists/${playlist_id}/tracks`,
        {
          uris: this.track_uris
        },
        {
          headers: {
            'Authorization': 'Bearer ' + this.access_token,
            'Content-Type': 'application/json'
          },
        }
      )
      console.log(res_add)
    },
    async getRecommend() {
      var limit_len = 20
      const res = await this.$axios.$get(
        'https://api.spotify.com/v1/recommendations',
        {
          params: {
            limit: limit_len,
            market: 'US',
            seed_genres: 'hip_hop',
            seed_tracks: '2x5Frs6ts14U4MSLrQQlXg'
          },
          headers: {
            'Authorization': 'Bearer ' + this.access_token,
          },
        }
      )
      for(let i = 0; i < limit_len; i++) {
        this.track_uris.push(res.tracks[i].uri)
      }
      this.tracks = res.tracks
      this.generated = true
      console.log(this.track_uris)
    }
  },
};
</script>

<style lang="scss">
$df_font: 'Josefin Sans', sans-serif;
$green: #1DB954;

  .main_wrapper {
    background-color: black;
    max-width: 100%;
    height: 100%;
    padding-bottom: 200px;
    h1 {
      padding-top: 200px;
      text-align: center;
      color: white;
      font-family: $df_font;
    }

    .inner_wrapper {
      max-width: 1280px;
      margin: 0 auto;
    }

    .login {
      color: white;
      margin: 0 auto;
      text-align: center;
      margin-top: 50px;
      p {
        font-family: $df_font;
        font-size: 24px;
        padding-bottom: 50px;
      }
      .btn {
        text-decoration: none;
        color: white;
        background-color: #1DB954;
        font-family: $green;
        padding: 7px 50px;
        border-radius: 20px;
        cursor: pointer;
      }
    }

    .logged {
      @extend .login;
      p {
        color: white;
        text-align: center;
      }

      .import {
        p {
            padding-bottom: 20px;
        }
        button {
          @extend .btn;
          border: none;
        }
      }

      .tracks {
        margin-top: 50px;
        margin-bottom: 30px;
        ul {
          width: 100%;
          padding: 0;
          li {
            a {
              text-decoration: none;
            }
            p {
              padding-bottom: 20px;
              font-size: 15px;
            }
            list-style: none;
          }
        }
      }

      .generate {
        margin-bottom: 30px;
        p {
          padding-bottom: 15px;
        }
        button {
          @extend .btn;
          border: none;
        }
      }

      .logout {
        @extend .btn;
      }
    }
  }
</style>

