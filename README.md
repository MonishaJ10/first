

app.comp.html
<router-outler/>

app.component.spec.ts
import { TestBed } from '@angular/core/testing';

import { AppComponent } from './app.component';

describe('AppComponent', () => {

beforeEach (async () => {

await TestBed.configureTestingModule({

imports: [AppComponent],

}).compileComponents();

});

it('should create the app', () => {

const fixture = TestBed.createComponent(AppComponent);

const app = fixture.componentInstance;

expect(app).toBeTruthy();

});

it('should have the 'uinextgen' title', () => {

const fixture = TestBed.createComponent (AppComponent);

const app = fixture.componentInstance;

expect(app.title).toEqual('uinextgen'); });

I

it('should render title', () => {

const fixture = TestBed.createComponent (AppComponent);

fixture.detectChanges();

const compiled fixture.native Element as HTMLElement;

expect(compiled.querySelector('h1')?.textContent).toContain ('Hello, uinextgen');

});

});

app.comp.ts

import { Component, OnInit } from '@angular/core';

import { CommonModule } from '@angular/common';

import { PrimeNGConfig } from 'primeng/api';

import { RouterOutlet } from '@angular/router';

@Component({

selector: 'app-root',

standalone: true,

imports: [CommonModule, RouterOutlet],

templateUrl: './app.component.html',

styleUrls: ['./app.component.css'],

})

export class AppComponent implements OnInit {

title = 'Recon-NextGen';

constructor(private primengConfig: PrimeNGConfig) {}

ngOnInit() {

this.primengConfig.ripple = true;

}

}

app.config.server.ts

import { mergeApplicationConfig, ApplicationConfig } from '@angular/core';

import { provideServerRendering } from '@angular/platform-server';

import { appConfig } from './app.config';

const serverConfig: ApplicationConfig = {

providers: [

provideServerRendering()

]

};

export const config = mergeApplicationConfig(appConfig, serverConfig);

app.config.ts
import { ApplicationConfig } from '@angular/core';

- import { provideRouter } from '@angular/router';

import { routes } from './app.routes';

import { provideClientHydration } from '@angular/platform-browser';

import { provideAnimations } from '@angular/platform-browser/animations';

import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';

import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {

providers: [provideRouter (routes), provideClientHydration(), provideAnimations(), provideAnimationsAsync() provideHttpClient()

};

app.routes.ts
import { Routes } from '@angular/router';

import { LoginComponent } from '../components/login/login.component';

import { HomeComponent } from '../components/home/home.component';

import { MatchComponent } from '../components/match/match.component';

import { LayoutComponent } from '../components/layout/layout.component';

export const routes: Routes = [

{ path:"", component: LoginComponent },

path:"",

component: LayoutComponent,

children: [

{

path: 'home',

component: HomeComponent

{

},

{

path: 'match',

component: MatchComponent },

]

}
];

components >home
home.component.css
main{
display: flex;
justify-content: center;
align-items: center;
position: absolute;
height: 100dvh;
width: 100%;
}

home.component.html
<main>
 Welcome to Home page
</main>

home.component.specs.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';

import { HomeComponent } from './home.component';

describe('HomeComponent', () => {

let component: HomeComponent;

let fixture: ComponentFixture<HomeComponent>;

beforeEach (async () => {

await TestBed.configureTestingModule({

imports: [HomeComponent]

})

.compileComponents();

fixture = TestBed.createComponent(HomeComponent);

component = fixture.componentInstance;

fixture.detectChanges();

});

it('should create', () => {

expect(component).toBeTruthy();

});

});

home.component.ts

import { Component } from '@angular/core';

@Component({

selector: 'app-home',

standalone: true,

imports: [],

templateUrl: './home.component.html',

styleUrl: './home.component.css'

})

export class HomeComponent {

}

components>layout 
footer
footer.component.css
.container{

display: flex;

position: absolute;

gap: 2em;

bottom: -1.4%;

left: 50%;

transform: translate(-50%,-50%);

| width: 100%;

height: 2.5%;

background-color: rgb(0 119 72);

color: blanchedalmond;

z-index: 1000;

}

.container marquee{

word-spacing: 0.55em;

font-size: 0.8rem;

display: flex;

align-items: center;

}

footer.component.html
 <div class="container">
<marquee behavior="scroll" direction="left">
Contact Dev team for support...
</marquee>
</div>

footer.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';

import { FooterComponent } from './footer.component';

describe('Footer Component', () => {

let component: FooterComponent;

let fixture: ComponentFixture<FooterComponent>;

beforeEach (async () => {

await TestBed.configureTestingModule({

imports: [FooterComponent]

})

.compileComponents();

fixture = TestBed.createComponent(Footer Component);

component = fixture.componentInstance;

fixture.detectChanges();

});

it('should create', () => {

expect(component).toBeTruthy();

});

});

footer.component.ts
import { Component } from '@angular/core';

@Component({

selector: 'app-footer',

standalone: true,

imports: [],

templateUrl: './footer.component.html',

styleUrl: './footer.component.css'

})

export class FooterComponent {

}

component>layout>header 
header.component.css
container{

display: flex;

position: absolute;

gap: 2em;

top: 3%;

left: 50%;

transform: translate(-50%,-50%);

width: 100%;

height: 6.5%;

box-shadow: 0 0 5px rgba(0, 0, 0, 0.14);

z-index: 1000;

border-top: rgb(0 119 72) 6px solid;

}

.container .logo img{

height: 78%;

background-color: #efeff05e;

I

}

.container .logo {

display: flex;

justify-content: center;

align-items: center;

padding-left: 1em;

}
.container.title{

display: flex;

justify-content: center;

align-items: center;

font-weight: 600;

letter-spacing: 0.06em;

font-size: large;

color:rgba(14, 89, 139, 0.76);

text-shadow: 1px 1px 10px rgba(13, 1, 234, 0.416);

font-style: italic;

}

header.component.html
<div class="container">

<div class="logo">

<!-- <img src="https://peoplemine.cib.echonet/PeopleMine/resources/images/BNP_PARIBAS_LOGO.png" alt="BNP LOGO"> -->
<img ngSrc="assets/LOGO.png" width="150" height="20" alt="BNP LOGO" priority>
</div>
<div class="title" >
NextGen Recon
</div>
</div>

header.component.spec.ts

import { ComponentFixture, TestBed } from '@angular/core/testing';
import { HeaderComponent } from './header.component';
describe('HeaderComponent', () => {
let component: HeaderComponent;
let fixture: ComponentFixture <HeaderComponent>;

beforeEach (async () => {

await TestBed.configureTestingModule({ imports: [HeaderComponent]

})

.compileComponents();

fixture = TestBed.createComponent(HeaderComponent);

component = fixture.componentInstance; fixture.detectChanges(); });

it('should create', () => { expect(component).toBeTruthy(); });

});

header.component.ts
import { Component } from '@angular/core';

import { NgOptimizedImage } from '@angular/common'

@Component({

selector: 'app-header',

standalone: true,

imports: [NgOptimizedImage],

templateUrl: './header.component.html',

styleUrl: './header.component.css'

})

export class HeaderComponent {

}

component>layout>sidebar

sidebar.component.css
.container {

position: fixed;

top: 10%;

right: 97%;

transform: translateY(-50%,-50%);

z-index: 2000;

display: flex;

justify-content: flex-end;

width: 100%;

}

.container > p-sidebar {

width: 5rem !important;

max-width: 90%;

}

:host::ng-deep .hello{

background-color: black !important;

border: none;

}

sidebar.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';

import { SidebarComponent } from './sidebar.component';

describe('SidebarComponent', () => {

let component: SidebarComponent;

let fixture: ComponentFixture<SidebarComponent>;

beforeEach (async () => {

await TestBed.configureTestingModule({

imports: [SidebarComponent]

})

compileComponents();

fixture = TestBed.createComponent(SidebarComponent);

component = fixture.componentInstance;

fixture.detectChanges();

});

it('should create', () => {

expect(component).toBeTruthy();

});

});

sidebar.component.ts

import { Component, inject, ViewChild } from '@angular/core';

import { SidebarModule } from 'primeng/sidebar';

import (ButtonModule } from 'primeng/button';

import { RippleModule } from 'primeng/ripple';

import { AvatarModule } from 'primeng/avatar';

import { StyleClassModule} from 'primeng/styleclass';

import { Sidebar } from 'primeng/sidebar';

import { Router } from '@angular/router';

import { CommonModule } from '@angular/common;

@Component({

selector: 'app-sidebar",

standalone: true,

imports: [SidebarModule, ButtonModule, RippleModule, Avatar Module, StyleClass Module, CommonModule],

templateurl: './sidebar.component.html',

styleurl: './sidebar.component.css',

})

export class Sidebar Component {

router-inject (Router)



@ViewChild('sidebarRef') sidebarRef!: Sidebar;

closeCallback(e:any): void {

this.sidebarRef.close(e);

}

sidebarVisible: boolean = false;

navigateToPath(path: String) {

if (path="match") (

this.router,navigateByUrl('match')

}

else if (path==="home"){

this.router,navigateByUrl('home')

}

}

}

sidebar.comp.html (didn't paste code)

layout.component.css
.layout{

display: flex;

flex-direction: column;

height: 100dvh;

width: 100%;

}

.main-area{

flex: 1;

display: flex;

}

layout.component.html
<div class="layout">

<app-header></app-header>

<div class="main-area">

<app-sidebar></app-sidebar>

| |

<router-outlet></router-outlet>

</div>

| <app-footer></app-footer>

</div>
layout.specs.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';

import { LayoutComponent } from './layout.component';

describe('LayoutComponent', () => {

let component: LayoutComponent;

let fixture: ComponentFixture<LayoutComponent>;

beforeEach (async () => {

await TestBed.configureTestingModule({

| imports: [LayoutComponent]

})

.compileComponents();

fixture = TestBed.createComponent (LayoutComponent);

component = fixture.componentInstance;

fixture.detectChanges();

});

it('should create', () => {

| expect(component).toBeTruthy();

});

});

layout.component.ts
import { Component } from '@angular/core';

import { HeaderComponent } from './header/header.component';

import { FooterComponent } from './footer/footer.component';

import { SidebarComponent } from './sidebar/sidebar.component';

import { Routeroutlet } from '@angular/router';

@Component({

selector: 'app-layout',

standalone: true,

imports: [HeaderComponent, Footer Component, Sidebar Component, RouterOutlet],

templateUrl: './layout.component.html',

styleurl: './layout.component.css'

})

export class LayoutComponent {

}

Src>components>login

login.component.css
.container {

background-color:#d5d3d22a;

/* background-color: #9c9cac72; */

height: 100dvh;

width: 100%;

display: flex;

justify-content: center;

align-items: center;

gap: 4dvw;

}

.Ptitle {

position: absolute;

color: rgba(6, 6, 6, 0.543);

bottom: 3%;

left: 1%;

font-weight: 500;

font-size: 1em;

font-style: italic;

}

I

.intro {

display: grid;

box-shadow: rgba(10, 10, 10, 0.05) 1px 1px 15px;

height: 80%;

width: 65%;

position: relative;

bottom: 0;

border-top-left-radius: 2dvh;

border-bottom-right-radius: 2dvh;

}
.box{

width: 80%;

height: 42%;

background-color: #ff060611;

display: flex;

align-items: center;

justify-content: center;

position: absolute;

/* border-radius:

2dvh;

*/

border-top-left-radius:

2dvh;

border-bottom-right-radius:

2dvh;

}
.switch {

box-shadow: rgba(10, 10, 10, 0.26) 1px 1px 20px;

border-top-left-radius: 5dvh;

border-bottom-right-radius: 5dvh;

/ border: 0.1dvh solid #111110; */

background: rgb(136, 136, 136);

/* background: -moz-linear-gradient(

edeg,

rgba(0, 0, 0, 1) 22%,

rgba(0, 0, 0, 1) 69%,

rgba(9, 9, 20, 1) 79%,

rgba(40, 38, 91, 1) 100%

); */

/* background: -webkit-linear-gradient(

edeg,

rgba(0, 0, 0, 1) 22%,

rgba(0, 0, 0, 1) 69%,

rgba(9, 9, 20, 1) 79%,

rgba(40, 38, 91, 1) 100% ); */

I

background: linear-gradient(

Odeg,

rgb(201, 201, 201) 22%, Orgba(0, 0, 0, 1) 69%, rgba (9, 9, 20, 1) 79%, rgb(0, 0, 0) 100%

);

filter: progid:DXImageTransform.Microsoft.gradient(startColorstr="#000000", endColorstr="#28265b", GradientType=1);

width: 25%;

position: relative;

/* border-radius: 2dvh; */

top: 4dvh;

}
Login .title {

display: flex;

justify-content: center;

padding: 0.5em 0;

color: white;

font-size: 1.1em;

font-style: italic;

}

Login.item {

color: white;

font-size: 0.8em;

display: flex;

padding: 1.2dvh;

justify-content: center;

}

.wrapper {

margin: 0 auto;

display: flex;

flex-direction: column;

gap: 4dvh;

align-items: center;

/* margin-top: em; */

/* margin-bottom: 1em; */

padding: 1em 1em;

I

}

.betterOutline:: placeholder {

font-style: italic;

}

betterOutline:focus {

--width: 3px;

--color: rgb(0 119 72);

/*--color: #020d71; */

outline: transparent;

box-shadow: 0px var(--width) var(--color);

}
.betteroutline {

font-family: inherit;

font-size: 0.8em;

color: inherit;

border: none;

border-radius: calc((1rem + 1.25rem * 2) / 2);

background-color:#333;

padding: 0.5rem 1.25rem;

box-shadow: none;

color: white;

transition: box-shadow 0.15s;

}

Login button {

border-radius: 50px;

background-color: rgb(1, 1, 1);

color: rgb(255, 253, 253);

cursor: pointer;

font-size: 0.9em;

padding: 0.7dvh 1dvw;

font-style: italic;

letter-spacing: 0.2dvw;

}

Login button:hover {

box-shadow:

rgb(10, 10, 10) 1px 1px 10px;

transition: ease-in 0.5s;

}

LoginSign {

cursor: pointer;

}

LoginSign:hover {

color: #2e0cefa0;

transition: 0.7s ease;

}

login.component.html
<app-header></app-header>

app-header

<app-footer></app-footer>

<div class="container">

<div class="Ptitle">Recon NextGen </div>

<div class="intro">

<div class="box" style="right: 17%; top: 5%;">Hyperlinks</div>

<div class="box" style="left: 17%;bottom: 5%; ">content Hyperlinks</div>

</div>

<div class="switch">

<div *ngIf="isLogin" class="Login">

<div class="title">Login</div>

<div class="item">Reconciliation</div>

<div class="item">Trade matching</div>

<div class="wrapper">

<input

type="text"

placeholder="UID"

class="betterOutline" [(ngModel)]="uid"

/>

<input

type="password"

placeholder="Password"

class="betterOutline" [(ngModel)]="password"

I

/>

</div>

<div class="item">

<button (click)="onLogin()">Submit</button>

</div>

<div class="item">(or)</div>

<div class="item" style="font-size: 0.8em; color: black;">

Login with

<span

class="LoginSign" (click)="onSwitch()"

>

&nbsp;SSO

</span>

</div>

</div>
<app-header></app-header>

app-header

<app-footer></app-footer>

<div class="container">

<div class="Ptitle">Recon NextGen </div>

<div class="intro">

<div class="box" style="right: 17%; top: 5%;">Hyperlinks</div>

<div class="box" style="left: 17%;bottom: 5%; ">content Hyperlinks</div>

</div>

<div class="switch">

<div *ngIf="isLogin" class="Login">

<div class="title">Login</div>

<div class="item">Reconciliation</div>

<div class="item">Trade matching</div>

<div class="wrapper">

<input

type="text"

placeholder="UID"

class="betterOutline" [(ngModel)]="uid"

/>

<input

type="password"

placeholder="Password"

class="betterOutline" [(ngModel)]="password"

I

/>

</div>

<div class="item">

<button (click)="onLogin()">Submit</button>

</div>

<div class="item">(or)</div>

<div class="item" style="font-size: 0.8em; color: black;">

Login with

<span

class="LoginSign" (click)="onSwitch()"

>

&nbsp;SSO

</span>

</div>

</div>
<div *ngIf="lisLogin" class="Login">

<div class="title">Sign Up</div>

<div class="item">Connect, create, and match</div>

<div class="item">All in one place</div>

<div class="item">

<button >SSO Login</button>

</div>

<div class="item" style="font-size: 0.8em; color: black;">

Have account?

<span

class="LoginSign" (click)="onSwitch()"

>

I

&nbsp; Login

</span>

</div>

</div>

</div>

<p-toast

</div>

/>

login.component.specs.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';

import { LoginComponent } from './login.component';

describe('LoginComponent', () => {

let component: LoginComponent;

let fixture: ComponentFixture<LoginComponent>;

beforeEach (async () => {

await TestBed.configureTestingModule({

imports: [LoginComponent]

})

.compileComponents();

fixture = TestBed.createComponent(LoginComponent);

component = fixture.componentInstance;

fixture.detectChanges();

});

it('should create', () => {

expect(component).toBeTruthy();

});

});

login.component.ts
import { CommonModule } from '@angular/common';

import { Component, inject } from '@angular/core';

import { HeaderComponent } from '../layout/header/header.component';

import { FooterComponent } from '../layout/footer/footer.component';

import { Router } from '@angular/router';

import { HttpClient } from '@angular/common/http';

import { FormsModule } from '@angular/forms';

import { ToastModule} from 'primeng/toast';

import { MessageService } from 'primeng/api';

@Component({

selector: 'app-login',

standalone: true,

imports: [CommonModule, HeaderComponent, Footer Component, Forms Module, ToastModule],

templateUrl: './login.component.html',

styleurl: './login.component.css',

providers: [MessageService]

})

export class LoginComponent {

constructor(private router: Router, private messageService: MessageService){ }

uid:string=""

I

password:string=""

http-inject(HttpClient)

isLogin: boolean=true

onSwitch(){

this.isLogin=this.isLogin

}

showError(message: any) {

this.messageService.add({ severity: 'contrast', summary: 'Error', detail: message });

}
onLogin(){

if(this.uid==='' || this.password===''){

this.showError("Fields cannot be empty")

}

else{

this.http.post('http://localhost:8080/api/login', {

uid: this.uid,

password: this.password

}).subscribe((res: any) => {

if (res.status === "success") {

this.router.navigateByUrl('home');

console.log("login success")

} else {

this.showError("Invalid Credentials")

}

});

}

}

NEXTGEN-UI/
├── .angular/
├── .vscode/
├── dist/
├── node_modules/
├── src/
│   ├── app/
│   │   ├── layout/
│   │   │   ├── layout.component.html
│   │   │   └── layout.component.scss
│   │   ├── login/
│   │   │   ├── login.component.html
│   │   │   └── login.component.scss
│   │   ├── match/
│   │   │   ├── match.component.html
│   │   │   └── match.component.scss
│   │   ├── app-routing.module.ts
│   │   └── app.module.ts
│   ├── assets/
│   ├── environments/
│   │   ├── environment.prod.ts
│   │   └── environment.ts
│   ├── favicon.ico
│   ├── index.html
│   ├── main.ts
│   ├── polyfills.ts
│   ├── styles.scss
│   └── test.ts
├── angular.json
├── package-lock.json
├── package.json
├── README.md
├── tsconfig.app.json
├── tsconfig.json
└── tsconfig.spec.json

.container {
  display: flex;
  align-items: center;
  justify-content: start;
  padding-left: 1em;
  width: 100%;
  height: 70px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.14);
  border-top: rgb(0 119 72) 6px solid;
  background-color: white;
  z-index: 1000;
}

.logo img {
  height: 50px; /* better aspect ratio */
  width: auto;
  background-color: transparent;
}

.logo {
  margin-right: 1em;
}

.title {
  font-weight: 600;
  letter-spacing: 0.06em;
  font-size: 1.5em;
  color: rgba(14, 89, 139, 0.76);
  text-shadow: 1px 1px 10px rgba(13, 1, 234, 0.2);
  font-style: italic;
  padding-top: 0.2em;
}
