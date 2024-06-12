---
title: "Securing Your Angular App: A Comprehensive Guide to Keycloak Integration"
seoTitle: "Keycloak Integration for Angular Security"
seoDescription: "Learn how to integrate Keycloak with your Angular app for secure authentication and authorization"
datePublished: Wed Jun 12 2024 01:34:21 GMT+0000 (Coordinated Universal Time)
cuid: clxb5r9io000109lchm2c3q6o
slug: securing-your-angular-app-a-comprehensive-guide-to-keycloak-integration
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/Lexcm-6FHRU/upload/1ef503d7e3b490b10f2bf5b226380083.jpeg
tags: authentication, angular, authorization, keycloak, docker-compose

---

One of the repetitive, complex, and important parts of an application is user authorization and authentication. There are many tools for this, like Firebase, Auth0, and Okta, or you can let users log in with third-party providers like Gmail, Microsoft, or social networks.

I have used Keycloak in web applications before and like the idea of centralizing authentication and authorization. This lets us focus on developing our application. In this guide, I will show you how to set up Keycloak with a simple Angular application to manage users.

### **But, what can Keycloak do for me?**

Keycloak is an open-source tool that makes authentication and authorization easier in apps and services. It helps manage users and includes features like single sign-on (SSO), identity federation, and role and permission management.

In simple terms, **Keycloak** helps you:

• **Authenticate Users**: Let users log in to your application securely.

• **Authorization**: Controls which parts of your application users can access based on their roles and permissions.

• **Identity Federation**: Connects with multiple identity providers (like Google, Facebook, etc.) so users can log in with their existing accounts.

• **Session Management**: Handles user sessions, including global logout and session expiration.

### Project running

This is the final result of the following guide.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718155861379/8e764a17-1519-47a7-b0fb-01ffe224e073.gif align="center")

### **Part 1. Make run Keycloak with Docker Compose**

First, let’s set up Keycloak using Docker Compose. Create a `docker-compose.yml` file with the following content:

```dockerfile
// docker-compose.yml
version: '3.8'

services:
  keycloak:
    image: quay.io/keycloak/keycloak:23.0.4
    container_name: keycloak
    ports:
      - "8080:8080"
    environment:
      - KEYCLOAK_ADMIN=${KEYCLOAK_ADMIN}
      - KEYCLOAK_ADMIN_PASSWORD=${KEYCLOAK_ADMIN_PASSWORD}
      - KC_DB=${KC_DB}
      - KC_DB_URL=${KC_DB_URL}
      - KC_DB_USERNAME=${POSTGRES_USER}
      - KC_DB_PASSWORD=${POSTGRES_PASSWORD}
      - KC_HOSTNAME=${KC_HOSTNAME}
    command: [ "start-dev", "--import-realm" ]

  postgres:
    image: postgres:latest
    container_name: postgres
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

And, a `.env` file at the same level as the `docker-compose.yml`:

```yaml
KEYCLOAK_ADMIN=admin
KEYCLOAK_ADMIN_PASSWORD=admin
KC_DB_URL=jdbc:postgresql://postgres:5432/keycloak
KC_HOSTNAME=localhost
KC_DB=postgres

POSTGRES_DB=keycloak
POSTGRES_USER=keycloak
POSTGRES_PASSWORD=password
```

Now, you can go to your terminal and do `docker compose up` and now you have a Keycloak up and running.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717867034124/02c8331d-2a74-4152-bc70-355e19a0b4f8.png align="center")

Now, we can access to our Keycloak using `http://localhost:8080` and configure our new client for our Angular app.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717867165048/45d6e3d5-3729-4acb-952e-f9933574d6c6.png align="center")

Clicking at `Administration Console` using the password added in the `.env` file (in our case is user: `admin`, password: `admin`)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717867426644/1ece2dd2-ebf2-4e38-9487-796f728e0f97.png align="center")

### Keycloak configuration

**Create a realm**: In Keycloak, a realm is an isolated space that contains everything needed to manage authentication and authorization for a set of applications or services. You can define users, roles, clients, identities, and security settings within a realm.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717883320354/6fa86120-6001-45b3-ba23-5c3d7c1614e1.png align="center")

**Create a client**: A client represents an application or service that uses Keycloak to manage authentication and authorization. Clients can be web apps, mobile apps, backend services, or any other entity that needs to authenticate users and manage their permissions. Our **angular app** will be a client of **Keycloak**.

Step 1: General settings:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717868180144/9efb694b-aac5-45d6-969f-d495b1ad0b31.png align="center")

Step 2: Capability config: We left it as is, in our basic demo, we don't need to change/add anything.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717882464633/f45d1d24-15fb-4720-ba60-38b80fdbe4a5.png align="center")

Step 3: Login settings: This one is important, lets to explain each input:

* **Root URL**: The root URL of our application, Keycloak uses this URL as the base for redirection after authentication.
    
* **Home URL**: It can be used by Keycloak to redirect users to the main page of your application after authentication, e.g., `http://localhost:8080/dashboard`
    
* **Valid redirect URIs**: Specify the URLs to which Keycloak is allowed to redirect after successful authentication. This is crucial to prevent malicious redirection attacks.
    
* **Valid post-logout redirect URIs**: Define the URLs to which Keycloak can redirect after logging out. This ensures that users are redirected to a safe and expected place after leaving the application.
    
* **Web origins**: Define which origins are allowed to make cross-origin requests to Keycloak. This is essential for your Angular application to communicate with Keycloak without being blocked due to CORS policies.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717882487574/03d05323-820c-4017-8419-03b0c3057739.png align="center")

**Create a test user for login**

For our Demo Angular App, we need to create a user. Go to Users and add a new user inside the realm we created.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717883539352/f514a6fe-8e0f-4bfc-8f7f-98a34d169aed.png align="center")

After creating it, we can add a password:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717883713593/18166796-9043-4211-94e3-3caea1af2269.png align="center")

And that's it. We are now ready to focus on our Angular app.

## Part 2. Integrate Keycloak with Angular

Assuming we already have an Angular application (Angular 18), we need to install these 2 libraries: `keycloak-js` and `keycloak-angular`:

```bash
npm install keycloak-js keycloak-angular 
// OR
yarn add keycloak-js keycloak-angular
```

What we will add to our Angular app

* Add the Keycloak config to our environment.ts file to use from there.
    

```typescript
// environment.development.ts
export const environment = {
  production: false,
  keycloak: {
    authority: 'http://localhost:8080', // Keycloak server
    redirectUri: 'http://localhost:4200', // redirect
    postLogoutRedirectUri: 'http://localhost:4200/logout', // post logout url
    realm: 'my-app',
    clientId: 'angular-app',
  }
};
```

* Initialize the Keycloak client and register it in `APP_INITIALIZER`
    

```typescript
// keycloak.factory.ts

import { KeycloakService } from 'keycloak-angular';
import { environment } from '../../environments/environment';

/**
 * Initializes Keycloak service.
 * @param {KeycloakService} keycloak - The Keycloak service instance.
 */
export function initializeKeycloak(keycloak: KeycloakService) {
  return async () => keycloak.init({
    config: {
      url: environment.keycloak.authority,
      realm: environment.keycloak.realm,
      clientId: environment.keycloak.clientId,
    },
    loadUserProfileAtStartUp: true,
    initOptions: {
      onLoad: 'check-sso',
      silentCheckSsoRedirectUri:
        window.location.origin + '/silent-check-sso.html',
      checkLoginIframe: false,
      redirectUri: environment.keycloak.redirectUri,
    },
  });
}
```

And register as a factory in `appConfig` with the `APP_INITIALIZER` token

```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(appRoutes),
    {
      provide: APP_INITIALIZER,
      useFactory: initializeKeycloak, // Our factory
      multi: true,
      deps: [KeycloakService],
    },
    KeycloakService
  ],
};
```

* Create the HTML file (`silent-check-sso.html`) for silent SSO verification
    

```xml
<html>
<body>
<script>
  parent.postMessage(location.href, location.origin);
</script>
</body>
</html>
```

This file is important to ensure everything works correctly. Make sure it is reachable. In my case, I placed it inside `/public/`. Check where your asset folder is located in `angular.json` or `project.json` (if you are using Nx). To double check if it is reachable `http://localhost:4200/silent-check-sso.html` in my case.

* Auth service to use `KeycloakService`
    

```typescript
// auth.service.ts

/**
 * Provides authentication services using Keycloak.
 *
 */
@Injectable({
  providedIn: 'root'
})
export class AuthService {
  readonly #keycloakService = inject(KeycloakService);

  /**
   * Redirects the user to the login page.
   *
   * @returns {Promise<void>} A promise that resolves when the user is redirected to the login page.
   */
  redirectToLoginPage(): Promise<void> {
    return this.#keycloakService.login();
  }

  /**
   * Retrieves the username of the currently authenticated user.
   *
   * @return {string} The username of the authenticated user.
   */
  get userName(): string {
    return this.#keycloakService.getUsername();
  }

  /**
   * Checks if the user is currently logged in.
   *
   * @return {boolean} - Returns true if the user is logged in, otherwise returns false.
   */
  isLoggedIn(): boolean {
    return this.#keycloakService.isLoggedIn();
  }

  /**
   * Logs out the user from the Keycloak service.
   *
   * @return {void} This method does not return any value.
   */
  logout(): void {
    this.#keycloakService.logout(environment.keycloak.postLogoutRedirectUri);
  }
}
```

* Create an **auth guard** to prevent users from visiting protected routes
    

```typescript
// auth.guard.ts
export const authGuard: CanActivateFn = () => {
  const authService = inject(AuthService);
  // Checks if the user is currently logged in.
  if (authService.isLoggedIn()) {
    // Allow go to the URL
    return true;
  }
  // Redirects the user to the login page.
  authService.redirectToLoginPage();
  return false;
};
```

* Use the authGuard in our `appRoutes` file:
    

```typescript
export const appRoutes: Route[] = [
  {
    path: '',
    pathMatch: 'full',
    component: MainPageComponent,
  },
  {
    path: 'public',
    component: PublicPageComponent,
  },
  {
    path: 'private',
    canActivate: [authGuard], // <-- only logged users
    component: PrivatePageComponent,
  },
  {
    path: 'logout',
    component: LogoutComponent,
  },
  {
    path: '404',
    component: NotFoundComponent,
  },
  {
    path: '**',
    redirectTo: '404',
  },
];
```

* Pages to verify the application flow
    
    * **MainPageComponent**: With a welcome page, which will show the available routes to access and a **Login** button that redirects us to Keycloak for authentication. \[[github](https://github.com/oidacra/angular-keycloak/blob/0fc378432b7073760dd39fd6b9ac75c09ff36657/src/app/pages/main/main-page.component.ts)\]
        
    * **PrivatePageComponent**: This is protected by the authGuard we created. If you are not logged in, it sends you to the login page. \[[github](https://github.com/oidacra/angular-keycloak/blob/0fc378432b7073760dd39fd6b9ac75c09ff36657/src/app/pages/private/private-page.component.ts)\]
        
    * **PublicPageComponent**: This is accessible to anyone, whether logged in or not. \[[github](https://github.com/oidacra/angular-keycloak/blob/0fc378432b7073760dd39fd6b9ac75c09ff36657/src/app/pages/public/public-page.component.ts)\]
        
    * **LogoutComponent**: This informs the user they have logged out. \[[github](https://github.com/oidacra/angular-keycloak/blob/0fc378432b7073760dd39fd6b9ac75c09ff36657/src/app/pages/logout/logout.component.ts)\]
        

This is the URL of the [repo](https://github.com/oidacra/angular-keycloak) of the project up and running if you want to download and run all together.

### Conclusions

Integrating Keycloak with your Angular app gives you a strong way to handle authentication and authorization. Keycloak takes care of security, so you can focus on building your app. It offers features like single sign-on, identity federation, and role-based access control, making the user experience secure and smooth.