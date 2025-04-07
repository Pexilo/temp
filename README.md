
Pour créer un tableau avec des colonnes fixes et défilantes en Angular 18, sans utiliser Angular Material, je vais vous montrer comment implémenter cette fonctionnalité en utilisant CSS et des directives personnalisées.

## Structure du projet

Commençons par créer les composants nécessaires:

```typescript
// table.component.ts
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-fixed-scroll-table',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './fixed-scroll-table.component.html',
  styleUrls: ['./fixed-scroll-table.component.scss']
})
export class FixedScrollTableComponent implements OnInit {
  // Données du tableau basées sur l'image
  tableData = {
    headers: ['Label', 'Total', 'Isabelle', 'Enfant 1', 'Enfant 2', 'Enfant 3'],
    rows: [
      { label: 'Transmission nette à la succession', total: '727 611 €', values: ['380 000 €', '173 806 €', '0 €', '0 €'] },
      { label: 'Capitaux décès nets', total: '0 €', values: ['0 €', '0 €', '0 €', '0 €'] },
      { label: 'Prélèvement (art. 990I-1 du CGI)', total: '0 €', values: ['0 €', '0 €', '0 €', '0 €'] },
      { label: 'Héritage net à la succession', total: '727 611 €', values: ['380 000 €', '173 806 €', '0 €', '0 €'] },
      { label: 'Parts réservées', total: '0 €', values: ['0 €', '0 €', '0 €', '0 €'] },
      { label: 'Héritage en pleine propriété', total: '0 €', values: ['0 €', '0 €', '0 €', '0 €'] },
      { label: 'Héritage en nue propriété', total: '0 €', values: ['0 €', '0 €', '0 €', '0 €'] },
      { label: 'Héritage en usufruit', total: '0 €', values: ['0 €', '0 €', '0 €', '0 €'] },
      { label: 'Valeur fiscale de l\'héritage', total: '0 €', values: ['0 €', '0 €', '0 €', '0 €'] },
      { label: 'Abattement personnel', total: '0 €', values: ['0 €', '0 €', '0 €', '0 €'] },
      { label: 'Part nette taxable', total: '0 €', values: ['0 €', '0 €', '0 €', '0 €'] },
      { label: 'Droit de succession', total: '506 667 €', values: ['32 389 €', '0 €', '0 €', '0 €'] },
    ]
  };

  ngOnInit(): void {}
}
```

## Template HTML

```html
<!-- fixed-scroll-table.component.html -->
<div class="table-container">
  <div class="fixed-columns">
    <table class="fixed-table">
      <thead>
        <tr>
          <th>{{ tableData.headers[0] }}</th>
          <th>{{ tableData.headers[1] }}</th>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let row of tableData.rows">
          <td>{{ row.label }}</td>
          <td>{{ row.total }}</td>
        </tr>
      </tbody>
    </table>
  </div>
  
  <div class="scrollable-columns">
    <table class="scrollable-table">
      <thead>
        <tr>
          <th *ngFor="let header of tableData.headers.slice(2)">{{ header }}</th>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let row of tableData.rows">
          <td *ngFor="let value of row.values">{{ value }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</div>
```

## CSS pour le tableau

```scss
// fixed-scroll-table.component.scss
.table-container {
  display: flex;
  width: 100%;
  border: 1px solid #ccc;
  overflow: hidden;
  background-color: #f0f0f0;
}

.fixed-columns {
  position: sticky;
  left: 0;
  z-index: 2;
  background-color: #f0f0f0;
}

.scrollable-columns {
  overflow-x: auto;
  flex-grow: 1;
}

table {
  border-collapse: collapse;
  width: 100%;
}

th, td {
  padding: 8px 12px;
  text-align: left;
  border-bottom: 1px solid #ddd;
  white-space: nowrap;
}

th {
  background-color: #f0f0f0;
  font-weight: bold;
}

tr:nth-child(even) {
  background-color: #f9f9f9;
}

.fixed-table th, .fixed-table td {
  border-right: 1px solid #ddd;
}

.scrollable-table {
  table-layout: fixed;
}

.scrollable-table th, .scrollable-table td {
  min-width: 120px;
}

// Assurer que les hauteurs des lignes correspondent entre les deux tableaux
.fixed-table tr, .scrollable-table tr {
  height: 40px;
}
```

## Directive pour synchroniser les hauteurs de ligne

Pour garantir que les hauteurs des lignes correspondent parfaitement entre les deux tableaux, nous pouvons créer une directive personnalisée:

```typescript
// sync-row-heights.directive.ts
import { AfterViewInit, Directive, ElementRef } from '@angular/core';

@Directive({
  selector: '[appSyncRowHeights]',
  standalone: true
})
export class SyncRowHeightsDirective implements AfterViewInit {
  constructor(private el: ElementRef) {}

  ngAfterViewInit() {
    const fixedTable = this.el.nativeElement.querySelector('.fixed-table');
    const scrollableTable = this.el.nativeElement.querySelector('.scrollable-table');
    
    if (!fixedTable || !scrollableTable) return;
    
    const fixedRows = fixedTable.querySelectorAll('tr');
    const scrollableRows = scrollableTable.querySelectorAll('tr');
    
    // Synchroniser les hauteurs des lignes
    for (let i = 0; i < Math.min(fixedRows.length, scrollableRows.length); i++) {
      const height = Math.max(fixedRows[i].offsetHeight, scrollableRows[i].offsetHeight);
      fixedRows[i].style.height = `${height}px`;
      scrollableRows[i].style.height = `${height}px`;
    }
  }
}
```

Ensuite, mettez à jour le composant pour utiliser cette directive:

```typescript
// table.component.ts (mise à jour)
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { SyncRowHeightsDirective } from './sync-row-heights.directive';

@Component({
  selector: 'app-fixed-scroll-table',
  standalone: true,
  imports: [CommonModule, SyncRowHeightsDirective],
  templateUrl: './fixed-scroll-table.component.html',
  styleUrls: ['./fixed-scroll-table.component.scss']
})
export class FixedScrollTableComponent implements OnInit {
  // ... reste du code inchangé
}
```

Et ajoutez la directive au template:

```html
<!-- fixed-scroll-table.component.html (mise à jour) -->
<div class="table-container" appSyncRowHeights>
  <!-- reste du code inchangé -->
</div>
```

## Personnalisation du style pour ressembler à l'image

Pour que le tableau ressemble davantage à celui de l'image, ajoutez ces styles supplémentaires:

```scss
// Ajoutez ces styles à fixed-scroll-table.component.scss
.table-container {
  font-family: Arial, sans-serif;
}

th {
  background-color: #a0a0a0;
  color: #000;
  font-weight: bold;
}

tr:nth-child(odd) {
  background-color: #e0e0e0;
}

tr:nth-child(even) {
  background-color: #f0f0f0;
}

// Style pour la dernière ligne (Droit de succession)
tr:last-child {
  background-color: #d0d0d0;
  font-weight: bold;
}

// Alignement des valeurs numériques
td:not(:first-child) {
  text-align: right;
}
```

Cette implémentation crée un tableau avec les deux premières colonnes fixes (Label et Total) et les autres colonnes défilantes horizontalement. La barre de défilement apparaît uniquement sous les colonnes défilantes, comme demandé, et non sous les colonnes fixes.

La directive `SyncRowHeightsDirective` assure que les hauteurs des lignes correspondent entre les deux tableaux, ce qui est essentiel pour maintenir l'alignement visuel.

Sources
[1] image.jpg https://pplx-res.cloudinary.com/image/upload/v1744028999/user_uploads/AfqQZKieCwYdrCF/image.jpg
[2] Searchable Data Table in Angular (NO Material) with SCSS, Useful ... https://dev.to/riapacheco/searchable-data-table-in-angular-no-material-with-scss-useful-directives-js-methods-and-virtual-scroll-1ilh
[3] AngularJS fixed header scrollable table directive https://www.pointblankdevelopment.com.au/blog/128/angularjs-fixed-header-scrollable-table-directive
[4] Scrollable table with fixed columns and header, with modern CSS https://stackoverflow.com/questions/46275555/scrollable-table-with-fixed-columns-and-header-with-modern-css
[5] How I implemented sticky columns in tables using directives - Angular! https://sreyaj.dev/implement-sticky-columns-using-directives-angular
[6] Scrolling in Angular Grid component - Syncfusion https://ej2.syncfusion.com/angular/documentation/grid/scrolling/scrolling
[7] Header div stays at top, vertical scrolling div below with scrollbar ... https://stackoverflow.com/questions/9677894/header-div-stays-at-top-vertical-scrolling-div-below-with-scrollbar-only-attach
[8] fixed header and scrollable body in table in html angular https://stackoverflow.com/questions/50640725/fixed-header-and-scrollable-body-in-table-in-html-angular
[9] Table with a fixed first column - DEV Community https://dev.to/nicolaserny/table-with-a-fixed-first-column-2c5b
[10] scrollbar-gutter - CSS: Cascading Style Sheets - MDN Web Docs https://developer.mozilla.org/en-US/docs/Web/CSS/scrollbar-gutter
[11] CSS-only bottom-anchored scrolling area - Kitty Giraudel https://kittygiraudel.com/2024/02/26/css-only-bottom-anchored-scrolling-area/
[12] Angular Material table Sizing/Scroll - Stack Overflow https://stackoverflow.com/questions/50824617/angular-material-table-sizing-scroll
[13] angular fixed column XY scrollable table - CodePen https://codepen.io/aifarfa/pen/jbMqba
[14] Angular Material Mat Table Vertical Scroll Example https://www.itsolutionstuff.com/post/angular-material-mat-table-vertical-scroll-exampleexample.html
[15] Angular Material Data Table: A Complete Example https://blog.angular-university.io/angular-material-data-table/
[16] Table | Angular Material https://material.angular.io/components/table
[17] Scrolling | Angular Material https://material.angular.io/cdk/scrolling
[18] Angular mat-table with sticky header & scrollable body - StackBlitz https://stackblitz.com/edit/angular-mat-table-with-sticky-header-scrollable-body
[19] [Table] Add better support for horizontal scrolling #8680 - GitHub https://github.com/angular/components/issues/8680
[20] Angular table with last columns fixed - Stack Overflow https://stackoverflow.com/questions/55684970/angular-table-with-last-columns-fixed
[21] Mastering Resizable Columns in Angular Table - DEV Community https://dev.to/chintanonweb/mastering-resizable-columns-in-angular-table-a-step-by-step-guide-for-developers-4f5n
[22] Table with sticky columns scroll should start after sticky ... - GitHub https://github.com/angular/material2/issues/15747
[23] Table with sticky columns - StackBlitz https://stackblitz.com/angular/nneoarrlekb?file=src%2Fapp%2Ftable-sticky-columns-example.html
[24] Angular Data Grid - Fixed and Sticky Columns - DevExtreme https://js.devexpress.com/Angular/Demos/WidgetsGallery/Demo/DataGrid/FixedAndStickyColumns/
[25] Managed Table Fixed Columns - AngularJS - SeanTheme https://seantheme.com/color-admin/admin/angularjs/
[26] Scroll Issue with fixed column settings - Handsontable Forum https://forum.handsontable.com/t/scroll-issue-with-fixed-column-settings/8105
[27] Table - fixed header, scrollable body, most robust/simple solution? https://stackoverflow.com/questions/1301049/table-fixed-header-scrollable-body-most-robust-simple-solution/1304286
[28] Table sticky headers/footer scrollbar position · Issue #13450 - GitHub https://github.com/angular/material2/issues/13450
[29] How to restore scrolling position in Angular? https://angular.love/angular-scroll-position-restoration
[30] CSS-only bottom-anchored scrolling area https://cssence.com/2024/bottom-anchored-scrolling-area/
[31] Scroll bar should not extend into sticky header for mat-table. #11924 https://github.com/angular/components/issues/11924
[32] Retain scroll position in Kendo UI for Angular | Telerik Forums https://www.telerik.com/forums/retain-scroll-position
[33] webkit-scrollbar - CSS: Cascading Style Sheets - MDN Web Docs https://developer.mozilla.org/en-US/docs/Web/CSS/::-webkit-scrollbar
[34] Angular Custom Scrollbar - NPM https://www.npmjs.com/package/ngx-scrollbar/v/4.1.0
[35] Independent scrolling panels with no body scroll (using just CSS) https://benfrain.com/independent-scrolling-panels-body-scroll-using-just-css/
[36] Pure CSS Horizontal Scrolling https://css-tricks.com/pure-css-horizontal-scrolling/
[37] Angular Table Component - PrimeNG https://primeng.org/table
[38] fixed header and scrollable body in table in html angular https://mdbootstrap.com/support/angular/fixed-header-and-scrollable-body-in-table-in-html-angular/
[39] Angular Material table Sizing/Scroll - Stack Overflow https://stackoverflow.com/questions/50824617/angular-material-table-sizing-scroll/58957768
[40] ng-table-virtual-scroll - NPM https://www.npmjs.com/package/ng-table-virtual-scroll
[41] Angular mat-table with sticky header (Fixed header) & scrollable body https://www.angularjswiki.com/material/mat-table-fixed-sticky-header/
[42] How to Create an HTML Table with a Fixed Left Column ... - W3docs https://www.w3docs.com/snippets/html/how-to-create-an-html-table-with-a-fixed-left-column-and-scrollable-body.html
[43] Column Fixing - Angular DataGrid - DevExtreme - DevExpress https://js.devexpress.com/Angular/Documentation/Guide/UI_Components/DataGrid/Columns/Column_Fixing/
[44] Angular: Infinite Scrolling - DEV Community https://dev.to/krivanek06/angular-infinite-scrolling-2jab
[45] Implement Scrollable table in Angular Material with 2 easy steps https://ampersandtutorials.com/angular/scrollable-table-in-angular-material/
