---
layout: post
title: I don’t want to use TestCase class anymore!
published: false
comments: true
excerpt_separator: <!--more-->
---

https://dev.to/koladev/django-rest-authentication-cmh


My Auth mixin for views

```
class ApiAuthMixin(ApiMixin):
    authentication_classes = (JWTAuthentication,)
    permission_classes = (IsAuthenticated,)
```



not mine: 
The Simple JWT library comes with two useful routes:

One to obtain access and refresh token (login) 'api/token/'
And another one to obtain a new access token using the refresh token 'api/token/refresh/'
It can actually do all the work but there are some issues here :
The login routes only return a pair of token
In the user registration flow, the user will be obliged to sign in again to retrieve the pair of tokens.



const axiosService = axios.create({
  baseURL: BACKEND_URL,
  headers: {
      'Content-Type': 'application/json',
  },
});

axios.defaults.baseURL = BACKEND_URL;

axiosService.interceptors.response.use((response) => {
  utils.decreaseAjaxCalls();
  // console.log("------------  Ajax pending", utils.getAjaxPendingCallsCount());
  return response;
}, (error) => {
  utils.decreaseAjaxCalls();
  // console.log("------------ error  Ajax pending", utils.getAjaxPendingCallsCount());
  if (error.response.status == 401) {
    authLogout();
  }
  return Promise.reject(error);
});


const refreshAuthLogic = (failedRequest) => {
  const refreshToken = localStorage.getItem('refreshToken');
  if (refreshToken !== null) {
    return axios.post(
      '/api/v1/auth/refresh/',
      {
        refresh: refreshToken
      }
    ).then((resp) => {
      console.log('Response: ',resp)
      const { access } = resp.data; 
      failedRequest.response.config.headers.Authorization = 'Bearer ' + access;
      store.dispatch(setTokens(resp.data))
    }).catch((err) => {
      console.log('Response error: ',err)
      if (err.response && err.response.status === 401) {
        moveToLoginScreen();
      }
    })
  }
}

const moveToLoginScreen = () => {
  store.dispatch(authLogout());
}

createAuthRefreshInterceptor(axiosService, refreshAuthLogic)

export default axiosService;
