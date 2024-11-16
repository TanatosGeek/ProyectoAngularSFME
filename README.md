## 1.-Creacion del Proyecto

Abrimos una terminal o consola y ejecutamos el siguiente comando para crear un nuevo
proyecto en Angular:

```bash
ng new nombreDelProyecto
```

Entramos a la carpeta con el comando
```bash
cd nombreDelProyecto
```
Despues configuras las opciones del proyecto a tus necesidades desde colores, animaciones, renderizacion , etc. (Por lo general aceptas todo pero no esta de mas leer).

Agregaremos Material Design a nuestro proyecto con el comando
```bash
ng add @angular/material
```

Inicializaremos el servidor
```bash
ng serve
```

## 2.-Crear el Servicio para Consumir la API

Generaremos un servicio que se encargará de consumir la API. En la
terminal, escribiremos:

```bash
ng generate service services/user
```
Despues nos iremos al archivo src/app/services/user.service.ts para configurar el API que estaremos usando de la siguiente forma.

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class UserService {

  private apiUrl = 'https://api.escuelajs.co/api/v1/users';  // <-Aqui va la url de tu API
  constructor(private http: HttpClient) { }
  getUsers(): Observable<any[]> {
    return this.http.get<any[]>(this.apiUrl);
  }
}
```

## 3.-Configurar HttpClient

Para poder realizar las consultas necesitaremos implementar HttpClient en el archivo src/app/app.config.ts el cual se configurara de la siguiente forma.

```typescript
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';
import { provideClientHydration } from '@angular/platform-browser';
import { provideHttpClient, withFetch } from '@angular/common/http';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideClientHydration(),
    provideHttpClient(withFetch()), //<- Aquí se añade provideHttpClient con withFetch()
    provideAnimationsAsync()
  ]
};
```
## 4.-Crear el Componente de la Tabla de Usuarios
Para crear nuestro componente el cual nos servira para ver el contenido de nuestra API 
```bash
ng generate component components/user-list
```

Tras generar nuestro componente ingresaremos al archivo 
src/app/components/user-list.component.ts al cual se le dara el sigiente formato

```typescript
import { AfterViewInit, Component, OnInit, viewChild } from '@angular/core';
import { MatPaginator } from '@angular/material/paginator';
import { MatSort } from '@angular/material/sort';
import { MatTableDataSource, MatTableModule } from '@angular/material/table';
import { UserService } from '../../services/user.service';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatToolbarModule } from '@angular/material/toolbar';
import { MatCardModule } from '@angular/material/card';

@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [
    MatPaginator,MatSort,
    MatTableModule,MatFormFieldModule,
    MatInputModule,
    MatToolbarModule,
    MatCardModule,
  ],
  templateUrl: './user-list.component.html',
  styleUrls: ['./user-list.component.css']
})
export class UserListComponent implements OnInit, AfterViewInit {
  displayedColumns: string[] = ['id', 'name', 'email', 'password' , 'role', 'avatar']; // <- Aqui agregaremos lo que quieras sacar de tu API,
   //asi que ve que es lo que te retorna tu API
  dataSource = new MatTableDataSource<any>([]); // Inicializa con un arreglo vacío

  readonly paginator = viewChild.required(MatPaginator);
  readonly sort = viewChild.required(MatSort);

  constructor(private userService: UserService) {}

  ngOnInit(): void {
    // Llama al servicio para obtener los usuarios y asignarlos al dataSource
    this.userService.getUsers().subscribe((data: any[]) => {
      this.dataSource.data = data; // Asigna los datos obtenidos a dataSource
    });
  }

  ngAfterViewInit() {
    this.dataSource.paginator = this.paginator();
    this.dataSource.sort = this.sort();
  }

  applyFilter(event: Event) {
    const filterValue = (event.target as HTMLInputElement).value;
    this.dataSource.filter = filterValue.trim().toLowerCase();

    if (this.dataSource.paginator) {
      this.dataSource.paginator.firstPage();
    }
  }
}
```

## 5.-Creación de la Vista para Mostrar los Datos en una Tabla

Ingresaremos al archivo src/app/components/user-list/user-list.component.html y le faremos el siguiente formato este es la parte visible para el suario, en el archivo .ts solo se recuperaron los datos y agregamos los componentes de Material  para su psoteriro uso en el .html en el cual le vamos a dar una vista bonita dado que estos datos se encuentra en un arreglo y no en una tabla de la siguiente forma.

```html
<div class="container">
  <mat-card class="user-card">
    <!-- Encabezado del Card -->
    <mat-toolbar color="primary">
      <span class="toolbar-title">User List</span>
    </mat-toolbar>

    <!-- Input para el filtro -->
    <div class="filter-container">
      <mat-form-field appearance="outline" class="filter-field">
        <mat-label>Filter</mat-label>
        <input matInput (input)="applyFilter($event)" placeholder="Search Users" />
      </mat-form-field>
    </div>

    <!-- Tabla de Usuarios -->
    <table mat-table [dataSource]="dataSource" matSort class="mat-elevation-z8 user-table">
      <!-- Columna de ID -->
      <ng-container matColumnDef="id">
        <th mat-header-cell *matHeaderCellDef mat-sort-header>ID</th>
        <td mat-cell *matCellDef="let user">{{ user.id }}</td>
      </ng-container>

      <!-- Columna de Nombre -->
      <ng-container matColumnDef="name">
        <th mat-header-cell *matHeaderCellDef mat-sort-header>Name</th>
        <td mat-cell *matCellDef="let user">{{ user.name }}</td>
      </ng-container>

      <!-- Columna de Email -->
      <ng-container matColumnDef="email">
        <th mat-header-cell *matHeaderCellDef mat-sort-header>Email</th>
        <td mat-cell *matCellDef="let user">{{ user.email }}</td>
      </ng-container>

      <!-- Columna de Contraseña -->
      <ng-container matColumnDef="password">
        <th mat-header-cell *matHeaderCellDef mat-sort-header>Password</th>
        <td mat-cell *matCellDef="let user"> {{ user.password }} </td>
      </ng-container>

      <!-- Columna de Rol -->
      <ng-container matColumnDef="role">
        <th mat-header-cell *matHeaderCellDef mat-sort-header>Role</th>
        <td mat-cell *matCellDef="let user">{{ user.role }}</td>
      </ng-container>

<!-- Columna de Avatar -->
      <ng-container matColumnDef="avatar">
        <th mat-header-cell *matHeaderCellDef mat-sort-header>Avatar</th>
        <td mat-cell *matCellDef="let user">
          <a [href]="user.avatar" target="_blank">
            <img [src]="user.avatar" alt="Avatar" />
          </a>
        </td>
      </ng-container>

      <!-- Filas y Encabezados -->
      <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
      <tr mat-row *matRowDef="let row; columns: displayedColumns;"></tr>
    </table>

    <!-- Paginador -->
    <mat-paginator [pageSize]="5" [pageSizeOptions]="[5, 10, 25]" aria-label="Select page"
      showFirstLastButtons></mat-paginator>
  </mat-card>
</div>
```

## Integrar el Componente en la Aplicación
Nos iremos al archivo  src/app/app.component.ts y agregaremos el componente para poderlo usar en el archivo src/app/app.component.html con el sigueinte formato 

```html
<app-user-list></app-user-list>
<router-outlet></router-outlet>
```

## Resultados
Este seria el resultado final de como se implemento la vista con Material, utilizando los componentes de paginacion y filtro
![image](https://github.com/user-attachments/assets/098e860f-d8c9-4e52-9381-7558036283f6)


## Preguntas


**1.- ¿Qué hace el método getUsers en este servicio?**
R=Lo que realiza es una solicitud en formato HTTP GET para poder obtener datos desde la API externa y devolver los resultados.

------------
**2.- ¿Por qué es necesario importar HttpClientModule?**
R=Este módulo ya no es necesario en angular 18, pero lo que hacia era realizar peticiones de GET, POST, PUT y DELETE y esto se vio remplazado por la importación de HttpCliente que hace lo mismo, pero adopta un enfoque mas moderno y uso solo de componentes.

------------
**3.- ¿Qué función cumple el método ngOnInit en el componente UserListComponent?**
R=Esto sirve para realizar la carga inicial de datos al momento que se crea el componente.

------------

**4.-¿Para qué sirve el bucle *ngFor en Angular? En el ejemplo**
R=Es el encargado de repetir las acciones en cada arreglo y crea una copia de ese bloque 

------------
**5. ¿Qué ventajas tiene el uso de servicios en Angular para el consumo de APIs?**
- Nos permite reutilizar código
- Código más sencillo
- Facilita la implementación de pruebas
- Mas fácil de mantener
- Optimiza el rendimiento

------------
**6. ¿Por qué es importante separar la lógica de negocio de la lógica de presentación?**
- Mejora el mantenimiento 
- Escalabilidad

------------
**7 ¿Qué otros tipos de datos o APIs podrías integrar en un proyecto como este?**
- Autentificación y automatización 
- Permite tomar datos de terceros 
- Implementar notificaciones
- Almacenar datos
- Datos en tiempo
