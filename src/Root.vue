<style>

html,
body {
    margin: 0;
    padding: 0;
    color: #444;
    font-family: arial, helvetica;
}

a {
    text-decoration: none;
    color: inherit;
}

a:hover,
a:visited {
    color: inherit;
}

.header {
    background-color: #666;
    overflow: hidden;
}

ul {
    list-style-type: none;
    margin: 0;
}

.header li {
    color: white;
    border-left: .1em solid #888;
    float: right;
    margin-top: .25em;
    margin-bottom: .25em;
}

.header li a {
    height: 100%;
    padding: .75em;
    display: block;
}

.v-link-active {
    background-color: #444;
}

.page-wrapper {
    width: 50%;
    margin: 0 25%;
}

input {
    display: block;
}

*[type="text"],
*[type="email"],
*[type="password"],
textarea {
    width: 100%;
    line-height: 2.5em;
    border: .2em solid #bbb;
    border-radius: .25em;
    margin: .5em;
    padding-left: 1em;
}

*[type="submit"] {
    border: none;
    border-radius: .25em;
    padding: .75em 2em;
    margin: .5em;
    font-weight: bold;
}

.center {
    text-align: center;
}

.article-link {
    margin: 1em;
}

.article-link a {
    background-color: #eee;
    border-radius: 1em;
    padding: 2em 4em;
    margin: 1em;
    text-align: center;
    font-weight: bold;
    display: block;
}

.article-link a:hover {
    background-color: #888;
    color: white;
}

.error {
    background-color: red;
    color: white;
    padding: .5em 1em;
}

.success {
    background-color: green;
    color: white;
    padding: .5em 1em;
}

.fade-in-out-transition {
    transition: all .3s ease;
    height: 1em;
}

.fade-in-out-enter,
.fade-in-out-leave {
    height: 0;
    opacity: 0;
}

</style>

<template>

<navigation :storage.sync="storage"></navigation>
<router-view :storage.sync="storage"></router-view>

</template>

<script>

var Navigation = require('./components/navigation');

export default {
    created: function() {
        var component = this;
        // make sure token is still valid
        this.$http.get('/api/v1/token', {
            email: window.localStorage.webUser
        }, {
            headers: {
                'Authorization': 'Token ' + window.localStorage.webToken
            }
        })
        .then(function(response) {
            console.log(response);
                if (response.data.token) {
                    component.storage.userLoggedIn = true;
                    // return a promise to be caught in a .then block
                    return this.$http.get('/api/v1/articles', // if still valid get articles
                    {
                        email: window.localStorage.webUser
                    },
                    {
                        headers: {
                            'Authorization': 'Token ' + window.localStorage.webToken
                        }
                    });
                } else {
                    delete window.localStorage.webToken;
                    delete window.localStorage.webUser;
                }
            })
            .then(function(response) {
                console.log(response);
                if (response.data && response.data.length > 0) {
                    component.$set('storage.user.articles', response.data); // reactive syntax

                    return this.$http.get('/api/v1/user', // then get user information
                    {
                        email: window.localStorage.webUser
                    },
                    {
                        headers: {
                            'Authorization': 'Token ' + window.localStorage.webToken
                        }
                    });
                }
            })
            .then(function(response) {
                console.log(response);
                if (response.data.email) {
                    component.$set('storage.user.info.name', response.data.name);
                    component.$set('storage.user.info.email', response.data.email);
                }
            })
            .catch(function(error) { // catches any errors in the chain
                delete window.localStorage.webToken;
                delete window.localStorage.webUser;
                console.log(error);
                console.log('logged out')
            });
    },
    data: function() { // global storage object accessed and mutated by all components
        return {
            storage: {
                user: {
                    info: {
                        name: ''
                    },
                    articles: []
                },
                userLoggedIn: false
            }
        };
    },
    components: {
        navigation: Navigation
    }
}

</script>
