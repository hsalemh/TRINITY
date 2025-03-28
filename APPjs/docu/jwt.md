Collecting workspace information

### Implémentation de JWT dans le Frontend et le Backend

#### Backend (NestJS)

Dans le backend, le JWT (JSON Web Token) est utilisé pour sécuriser les endpoints et gérer l'authentification des utilisateurs. Voici comment il est implémenté :

1. **Module d'Authentification (`auth.module.ts`)**:
   - Le module d'authentification configure les services et les stratégies nécessaires pour l'authentification JWT.

```typescript
import { Module, forwardRef } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { AuthService } from './auth.service';
import { UsersModule } from '../users/users.module';
import { JwtStrategy } from './jwt/jwt.strategy';
import { AuthController } from './auth.controller';

@Module({
  imports: [
    forwardRef(() => UsersModule),
    PassportModule,
    JwtModule.register({
      secret: 'Epi-@tech242424@@epitech',
      signOptions: { expiresIn: '60m' },
    }),
  ],
  providers: [AuthService, JwtStrategy],
  controllers: [AuthController],
  exports: [AuthService],
})
export class AuthModule {}
```

2. **Service d'Authentification (`auth.service.ts`)**:
   - Le service d'authentification contient la logique pour vérifier les informations d'identification de l'utilisateur et générer un token JWT.

```typescript
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { UsersService } from '../users/users.service';
import { JwtService } from '@nestjs/jwt';
import * as bcrypt from 'bcryptjs';

@Injectable()
export class AuthService {
  constructor(
    private readonly usersService: UsersService,
    private readonly jwtService: JwtService,
  ) {}

  async login(email: string, password: string) {
    const user = await this.usersService.findOneByEmail(email);
    if (!user || !(await bcrypt.compare(password, user.password))) {
      throw new UnauthorizedException('Invalid credentials');
    }
    const payload = { email: user.email, sub: user.id, role: user.role };
    return {
      access_token: this.jwtService.sign(payload),
    };
  }
}
```

3. **Stratégie JWT (`jwt.strategy.ts`)**:
   - La stratégie JWT est utilisée pour valider le token JWT et extraire les informations de l'utilisateur.

```typescript
import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: 'Epi-@tech242424@@epitech',
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub, email: payload.email, role: payload.role };
  }
}
```

4. **Contrôleur d'Authentification (`auth.controller.ts`)**:
   - Le contrôleur d'authentification gère les requêtes HTTP pour l'authentification, comme la connexion.

```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { AuthService } from './auth.service';

@Controller('auth')
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post('login')
  async login(@Body() body: { email: string, password: string }) {
    return this.authService.login(body.email, body.password);
  }
}
```

#### Frontend (Vue.js)

Dans le frontend, le JWT est utilisé pour authentifier les utilisateurs et sécuriser les requêtes API. Voici comment il est implémenté :

1. **Formulaire de Connexion (`LoginForm.vue`)**:
   - Le formulaire de connexion permet à l'utilisateur de saisir ses informations d'identification et d'obtenir un token JWT.

```vue
<template>
  <form @submit.prevent="login">
    <input v-model="email" type="email" placeholder="Email" required />
    <input v-model="password" type="password" placeholder="Password" required />
    <button type="submit">Login</button>
  </form>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      email: '',
      password: ''
    };
  },
  methods: {
    async login() {
      try {
        const response = await axios.post('http://localhost:4000/auth/login', {
          email: this.email,
          password: this.password
        });
        localStorage.setItem('token', response.data.access_token);
        this.$router.push('/profile');
      } catch (error) {
        console.error('Login failed:', error);
      }
    }
  }
};
</script>
```

2. **Intercepteur Axios**:
   - Un intercepteur Axios est utilisé pour ajouter automatiquement le token JWT aux en-têtes des requêtes API.

```javascript
import axios from 'axios';

axios.interceptors.request.use(
  config => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  error => {
    return Promise.reject(error);
  }
);
```

3. **Route Protégée**:
   - Les routes protégées vérifient si l'utilisateur est authentifié avant de permettre l'accès.

```javascript
import Vue from 'vue';
import Router from 'vue-router';
import HomeView from '@/views/HomeView.vue';
import ProfileView from '@/views/ProfileView.vue';
import LoginView from '@/views/LoginView.vue';

Vue.use(Router);

const router = new Router({
  routes: [
    { path: '/', component: HomeView },
    { path: '/login', component: LoginView },
    {
      path: '/profile',
      component: ProfileView,
      meta: { requiresAuth: true }
    }
  ]
});

router.beforeEach((to, from, next) => {
  const token = localStorage.getItem('token');
  if (to.matched.some(record => record.meta.requiresAuth) && !token) {
    next('/login');
  } else {
    next();
  }
});

export default router;
```

### Explication pour la Soutenance

1. **Backend**:
   - **Module d'Authentification**: Configure les services et les stratégies nécessaires pour l'authentification JWT.
   - **Service d'Authentification**: Vérifie les informations d'identification de l'utilisateur et génère un token JWT.
   - **Stratégie JWT**: Valide le token JWT et extrait les informations de l'utilisateur.
   - **Contrôleur d'Authentification**: Gère les requêtes HTTP pour l'authentification, comme la connexion.

2. **Frontend**:
   - **Formulaire de Connexion**: Permet à l'utilisateur de saisir ses informations d'identification et d'obtenir un token JWT.
   - **Intercepteur Axios**: Ajoute automatiquement le token JWT aux en-têtes des requêtes API.
   - **Route Protégée**: Vérifie si l'utilisateur est authentifié avant de permettre l'accès aux routes protégées.

En résumé, le JWT est utilisé pour sécuriser les endpoints dans le backend et pour authentifier les utilisateurs dans le frontend. Le token JWT est généré par le backend lors de la connexion et est stocké dans le localStorage du navigateur. Ce token est ensuite utilisé pour authentifier les requêtes API dans le frontend.

Similar code found with 1 license type