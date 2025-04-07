Pour forcer le défilement horizontal dans votre tableau Angular et n'afficher que les colonnes "Label", "Total", "Isabelle" et "Enfant 1" par défaut, tout en rendant "Enfant 2", "Enfant 3" et les autres colonnes défilables, vous devez modifier votre implémentation actuelle.

## Modification de la structure du tableau

Vous devez modifier la structure pour avoir trois sections :
1. Les deux premières colonnes fixes (Label et Total)
2. Les deux colonnes suivantes semi-fixes (Isabelle et Enfant 1)
3. Les colonnes restantes défilables (Enfant 2, Enfant 3, etc.)

Voici comment adapter votre code :

```html
<div class="table-container">
  <!-- Colonnes fixes (Label et Total) -->
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
  
  <!-- Colonnes semi-fixes (Isabelle et Enfant 1) -->
  <div class="semi-fixed-columns">
    <table class="semi-fixed-table">
      <thead>
        <tr>
          <th>{{ tableData.headers[2] }}</th>
          <th>{{ tableData.headers[3] }}</th>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let row of tableData.rows">
          <td>{{ row.values[0] }}</td>
          <td>{{ row.values[1] }}</td>
        </tr>
      </tbody>
    </table>
  </div>
  
  <!-- Colonnes défilables (Enfant 2, Enfant 3, etc.) -->
  <div class="scrollable-columns">
    <table class="scrollable-table">
      <thead>
        <tr>
          <th *ngFor="let header of tableData.headers.slice(4)">{{ header }}</th>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let row of tableData.rows">
          <td *ngFor="let value of row.values.slice(2)">{{ value }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</div>
```

## CSS pour forcer le défilement

Pour forcer le défilement, vous devez définir une largeur fixe pour le conteneur des colonnes défilables et utiliser `overflow-x: auto` :

```scss
.table-container {
  display: flex;
  width: 100%;
  border: 1px solid #ccc;
  overflow: hidden;
}

.fixed-columns, .semi-fixed-columns {
  position: sticky;
  z-index: 2;
  background-color: #f0f0f0;
}

.fixed-columns {
  left: 0;
}

.semi-fixed-columns {
  border-right: 2px solid #aaa; /* Séparation visuelle entre les colonnes fixes et défilables */
}

.scrollable-columns {
  overflow-x: auto;
  width: 200px; /* Largeur fixe pour forcer le défilement */
  flex-shrink: 0; /* Empêche la réduction de la largeur */
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

.scrollable-table th, .scrollable-table td {
  min-width: 120px; /* Largeur minimale pour chaque colonne défilable */
}

/* Assurer que les hauteurs des lignes correspondent entre les tableaux */
.fixed-table tr, .semi-fixed-table tr, .scrollable-table tr {
  height: 40px;
}
```

## Directive pour synchroniser les hauteurs

Mettez à jour votre directive pour synchroniser les hauteurs entre les trois tableaux :

```typescript
import { AfterViewInit, Directive, ElementRef } from '@angular/core';

@Directive({
  selector: '[appSyncRowHeights]',
  standalone: true
})
export class SyncRowHeightsDirective implements AfterViewInit {
  constructor(private el: ElementRef) {}

  ngAfterViewInit() {
    const fixedTable = this.el.nativeElement.querySelector('.fixed-table');
    const semiFixedTable = this.el.nativeElement.querySelector('.semi-fixed-table');
    const scrollableTable = this.el.nativeElement.querySelector('.scrollable-table');
    
    if (!fixedTable || !semiFixedTable || !scrollableTable) return;
    
    const fixedRows = fixedTable.querySelectorAll('tr');
    const semiFixedRows = semiFixedTable.querySelectorAll('tr');
    const scrollableRows = scrollableTable.querySelectorAll('tr');
    
    // Synchroniser les hauteurs des lignes
    for (let i = 0; i < Math.min(fixedRows.length, semiFixedRows.length, scrollableRows.length); i++) {
      const height = Math.max(
        fixedRows[i].offsetHeight, 
        semiFixedRows[i].offsetHeight, 
        scrollableRows[i].offsetHeight
      );
      fixedRows[i].style.height = `${height}px`;
      semiFixedRows[i].style.height = `${height}px`;
      scrollableRows[i].style.height = `${height}px`;
    }
  }
}
```

Cette approche permet d'afficher les quatre premières colonnes (Label, Total, Isabelle, Enfant 1) de manière fixe, tandis que les colonnes restantes (Enfant 2, Enfant 3, etc.) seront défilables horizontalement. La barre de défilement apparaîtra uniquement sous les colonnes défilables.

Pour forcer l'apparition de la barre de défilement même si toutes les colonnes pourraient tenir dans l'espace disponible, la largeur fixe de `.scrollable-columns` est essentielle. Vous pouvez ajuster cette valeur selon vos besoins.

Sources
[1] fixed header and scrollable body in table in html angular https://mdbootstrap.com/support/angular/fixed-header-and-scrollable-body-in-table-in-html-angular/
[2] Angular Table Component - PrimeNG https://primeng.org/table
[3] Table | Angular Material https://material.angular.io/components/table
[4] angular-scrollable-table - NPM https://www.npmjs.com/package/angular-scrollable-table
[5] fixed header and scrollable body in table in html angular https://stackoverflow.com/questions/50640725/fixed-header-and-scrollable-body-in-table-in-html-angular
[6] Mastering Resizable Columns in Angular Table - DEV Community https://dev.to/chintanonweb/mastering-resizable-columns-in-angular-table-a-step-by-step-guide-for-developers-4f5n
[7] Mat-Table : Add vertical scrolling only on mat rows and not ... - GitHub https://github.com/angular/components/issues/10698
[8] Scroll bar should not extend into sticky header for mat-table. #11924 https://github.com/angular/components/issues/11924
[9] Table | Angular Material https://v8.material.angular.io/components/table/overview
[10] Angular Material table Sizing/Scroll - Stack Overflow https://stackoverflow.com/questions/50824617/angular-material-table-sizing-scroll
[11] Angular Material table Sizing/Scroll - Stack Overflow https://stackoverflow.com/questions/50824617/angular-material-table-sizing-scroll/58957768
[12] Scrolling | Angular Material https://material.angular.io/cdk/scrolling
[13] angular fixed column XY scrollable table - CodePen https://codepen.io/aifarfa/pen/jbMqba
[14] Can we please have a simpler solution for scrollable options https://www.telerik.com/forums/can-we-please-have-a-simpler-solution-for-scrollable-options
[15] Angular mat-table with sticky header & scrollable body - StackBlitz https://stackblitz.com/edit/angular-mat-table-with-sticky-header-scrollable-body
[16] Angular Material table style for horizontal scroll : r/Angular2 - Reddit https://www.reddit.com/r/Angular2/comments/122kxs5/angular_material_table_style_for_horizontal_scroll/
[17] bug(table): Sticky columns table can't fixed the width of ... - GitHub https://github.com/angular/components/issues/25396
[18] Implement Scrollable table in Angular Material with 2 easy steps https://ampersandtutorials.com/angular/scrollable-table-in-angular-material/
[19] Scroll Issue with fixed column settings - Handsontable Forum https://forum.handsontable.com/t/scroll-issue-with-fixed-column-settings/8105
