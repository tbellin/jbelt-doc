# jbelt-doc
Documentazione Jbelt 

Informazioni di base sullo sviluppo delle soluzioni Jbelt

E' rimandato ad una cartella doc e/o repository nella forma Jbelt-<modulo>-doc per ogni progetto specifico

## Data base PDM x Creo Parametric

Prima fase lo sviluppo di undatabase generico adottabile per supportare la base dati di Creo Parametric

La prima bozza è stata richiesta a ChatGPT 

> Per una migliore organizzazione e gestione dei file, possiamo includere una struttura di cartelle 
> che consente di associare le diverse entità a specifiche cartelle. 
> Questo consente di categorizzare i file in librerie, progetti, file di ufficio, ecc.

### Schema del Database Aggiornato con Cartelle

#### Tabelle Principali

1. **Progetti (Projects)**

```sql
CREATE TABLE Projects (
    ProjectID INT PRIMARY KEY AUTO_INCREMENT,
    ProjectName VARCHAR(255) NOT NULL,
    Description TEXT,
    StartDate DATE,
    EndDate DATE
);
```

2. **Modelli (Models)**

```sql
CREATE TABLE Models (
    ModelID INT PRIMARY KEY AUTO_INCREMENT,
    ProjectID INT,
    ModelName VARCHAR(255) NOT NULL,
    ModelType ENUM('Part', 'Assembly') NOT NULL,
    FilePath VARCHAR(255),
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (ProjectID) REFERENCES Projects(ProjectID)
);
```

3. **Disegni (Drawings)**

```sql
CREATE TABLE Drawings (
    DrawingID INT PRIMARY KEY AUTO_INCREMENT,
    ModelID INT,
    DrawingName VARCHAR(255) NOT NULL,
    FilePath VARCHAR(255),
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (ModelID) REFERENCES Models(ModelID)
);
```

4. **Family Table (FamilyTables)**

```sql
CREATE TABLE FamilyTables (
    FamilyTableID INT PRIMARY KEY AUTO_INCREMENT,
    ModelID INT,
    InstanceName VARCHAR(255),
    Parameters JSON,
    FOREIGN KEY (ModelID) REFERENCES Models(ModelID)
);
```

5. **Componenti dell'Assieme (AssemblyComponents)**

```sql
CREATE TABLE AssemblyComponents (
    AssemblyComponentID INT PRIMARY KEY AUTO_INCREMENT,
    ParentAssemblyID INT,
    ChildModelID INT,
    Quantity INT,
    Status ENUM('Suppressed', 'Placed', 'Fixed'),
    FOREIGN KEY (ParentAssemblyID) REFERENCES Models(ModelID),
    FOREIGN KEY (ChildModelID) REFERENCES Models(ModelID)
);
```

#### Tabelle degli Attributi

6. **Tipi di Attributo (AttributeTypes)**

```sql
CREATE TABLE AttributeTypes (
    AttributeTypeID INT PRIMARY KEY AUTO_INCREMENT,
    AttributeName VARCHAR(255) NOT NULL,
    AttributeDataType ENUM('String', 'Integer', 'Real', 'Boolean') NOT NULL
);
```

7. **Libreria dei Parametri (ParameterLibrary)**

```sql
CREATE TABLE ParameterLibrary (
    ParameterID INT PRIMARY KEY AUTO_INCREMENT,
    AttributeTypeID INT,
    ParameterName VARCHAR(255) NOT NULL,
    FOREIGN KEY (AttributeTypeID) REFERENCES AttributeTypes(AttributeTypeID)
);
```

8. **Parametri del Modello (ModelParameters)**

```sql
CREATE TABLE ModelParameters (
    ModelParameterID INT PRIMARY KEY AUTO_INCREMENT,
    ModelID INT,
    ParameterID INT,
    StringValue VARCHAR(255),
    IntegerValue INT,
    RealValue DECIMAL(10, 2),
    BooleanValue BOOLEAN,
    FOREIGN KEY (ModelID) REFERENCES Models(ModelID),
    FOREIGN KEY (ParameterID) REFERENCES ParameterLibrary(ParameterID)
);
```

9. **Parametri del Disegno (DrawingParameters)**

```sql
CREATE TABLE DrawingParameters (
    DrawingParameterID INT PRIMARY KEY AUTO_INCREMENT,
    DrawingID INT,
    ParameterID INT,
    StringValue VARCHAR(255),
    IntegerValue INT,
    RealValue DECIMAL(10, 2),
    BooleanValue BOOLEAN,
    FOREIGN KEY (DrawingID) REFERENCES Drawings(DrawingID),
    FOREIGN KEY (ParameterID) REFERENCES ParameterLibrary(ParameterID)
);
```

#### Tabelle dei Formati e dei Layout

10. **Formati (SheetFormats)**

```sql
CREATE TABLE SheetFormats (
    SheetFormatID INT PRIMARY KEY AUTO_INCREMENT,
    FormatName VARCHAR(50) NOT NULL,
    Width DECIMAL(10, 2) NOT NULL,
    Height DECIMAL(10, 2) NOT NULL
);
```

11. **Foglio di Lavoro (WorkSheets)**

```sql
CREATE TABLE WorkSheets (
    WorkSheetID INT PRIMARY KEY AUTO_INCREMENT,
    DrawingID INT,
    SheetFormatID INT,
    SheetNumber INT,
    FilePath VARCHAR(255),
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (DrawingID) REFERENCES Drawings(DrawingID),
    FOREIGN KEY (SheetFormatID) REFERENCES SheetFormats(SheetFormatID)
);
```

12. **Layout (Layouts)**

```sql
CREATE TABLE Layouts (
    LayoutID INT PRIMARY KEY AUTO_INCREMENT,
    LayoutName VARCHAR(255) NOT NULL,
    Description TEXT
);
```

13. **Parametri del Layout (LayoutParameters)**

```sql
CREATE TABLE LayoutParameters (
    LayoutParameterID INT PRIMARY KEY AUTO_INCREMENT,
    LayoutID INT,
    ParameterID INT,
    ParameterValue VARCHAR(255),
    FOREIGN KEY (LayoutID) REFERENCES Layouts(LayoutID),
    FOREIGN KEY (ParameterID) REFERENCES ParameterLibrary(ParameterID)
);
```

14. **Associazione Layout-Modello (LayoutModelAssociations)**

```sql
CREATE TABLE LayoutModelAssociations (
    LayoutModelAssociationID INT PRIMARY KEY AUTO_INCREMENT,
    LayoutID INT,
    ModelID INT,
    FOREIGN KEY (LayoutID) REFERENCES Layouts(LayoutID),
    FOREIGN KEY (ModelID) REFERENCES Models(ModelID)
);
```

#### Tabelle dei File

15. **Tipi di File (FileTypes)**

```sql
CREATE TABLE FileTypes (
    FileTypeID INT PRIMARY KEY AUTO_INCREMENT,
    FileTypeName VARCHAR(50) NOT NULL
);
```

16. **File (Files)**

```sql
CREATE TABLE Files (
    FileID INT PRIMARY KEY AUTO_INCREMENT,
    FileName VARCHAR(255) NOT NULL,
    Description TEXT,
    Version VARCHAR(50),
    FileTypeID INT,
    FilePath VARCHAR(255),
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (FileTypeID) REFERENCES FileTypes(FileTypeID)
);
```

17. **Cartelle (Folders)**

```sql
CREATE TABLE Folders (
    FolderID INT PRIMARY KEY AUTO_INCREMENT,
    FolderName VARCHAR(255) NOT NULL,
    ParentFolderID INT,
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (ParentFolderID) REFERENCES Folders(FolderID)
);
```

18. **Associazione File-Cartella (FolderFileAssociations)**

```sql
CREATE TABLE FolderFileAssociations (
    FolderFileAssociationID INT PRIMARY KEY AUTO_INCREMENT,
    FolderID INT,
    FileID INT,
    FOREIGN KEY (FolderID) REFERENCES Folders(FolderID),
    FOREIGN KEY (FileID) REFERENCES Files(FileID)
);
```

19. **Associazione Entità-File (EntityFileAssociations)**

```sql
CREATE TABLE EntityFileAssociations (
    EntityFileAssociationID INT PRIMARY KEY AUTO_INCREMENT,
    FileID INT,
    EntityType ENUM('Project', 'Model', 'Drawing', 'WorkSheet') NOT NULL,
    EntityID INT,
    FOREIGN KEY (FileID) REFERENCES Files(FileID)
);
```

#### Tabelle Ausiliarie

20. **Quote (Dimensions)**

```sql
CREATE TABLE Dimensions (
    DimensionID INT PRIMARY KEY AUTO_INCREMENT,
    DrawingID INT,
    DimensionType VARCHAR(255),
    DimensionValue DECIMAL(10, 2),
    Unit VARCHAR(50),
    FOREIGN KEY (DrawingID) REFERENCES Drawings(DrawingID)
);
```

21. **Simboli (Symbols)**

```sql
CREATE TABLE Symbols (
    SymbolID INT PRIMARY KEY AUTO_INCREMENT,
    DrawingID INT,
    SymbolName VARCHAR(255),
    SymbolType VARCHAR(255),
    PositionX DECIMAL(10, 2),
    PositionY DECIMAL(10, 2),
    FOREIGN KEY (DrawingID) REFERENCES Drawings(DrawingID)
);
```

22. **Revisioni (Revisions)**

```sql
CREATE TABLE Revisions (
    RevisionID INT PRIMARY KEY AUTO_INCREMENT,
    ModelID INT,
    RevisionNumber INT,
    ChangeDescription TEXT,
    RevisionDate DATE,
    FOREIGN KEY (ModelID) REFERENCES Models(ModelID)
);
```

23. **Utenti (Users)**

```sql
CREATE TABLE Users (
    UserID INT PRIMARY KEY AUTO_INCREMENT,
    UserName VARCHAR(255) NOT NULL,
    Email VARCHAR(255) NOT NULL,
    PasswordHash VARCHAR(255) NOT NULL,
    Role ENUM('Admin', 'Designer', 'Viewer') NOT NULL
);
```

24. **Permessi (Permissions)**

```sql
CREATE TABLE Permissions (
    PermissionID INT PRIMARY KEY AUTO_INCREMENT,
    UserID INT,
    ProjectID INT,
    AccessLevel ENUM('Read', 'Write', 'Admin') NOT NULL,
    FOREIGN KEY (UserID) REFERENCES Users(UserID),
    FOREIGN KEY (ProjectID) REFERENCES Projects(ProjectID)
);
```

### Descrizione della Struttura

1. **Progetti (Projects)**: Contiene informazioni generali sui progetti.
2. **Modelli (Models)**: Gestisce i modelli CAD, specificando se sono parti o assiemi.
3. **Disegni (Drawings)**: Contiene informazioni sui disegni CAD associati ai modelli.
4. **Family Table (FamilyTables)**: Contiene informazioni sulle varianti dei modelli, definite nelle Family Table di Creo.
5. **Componenti dell'Assieme (AssemblyComponents)**

> Ottimo lavoro, devo valutarlo ma mi pare una buona soluzione. Potresi formulare un file in formato JDL per caricarlo su Jhipster.
> Tieni conto che users potrebbe essere una estensione di User di Systema, abbiamo gia applicato questa formula con la tabella Anagrafic 
> e tutti fanno riferimenti ad Anagrafic e anagrafic si collega alla tabella User di Jhipster


Certamente! Ecco un esempio di file JDL che include le tabelle descritte, considerando anche la possibile estensione della tabella `User` di JHipster con una tabella `Anagrafic` che funge da riferimento centrale:

```jdl
entity User extends User {
  firstName String,
  lastName String,
  email String,
  password String
}

entity Anagrafic {
  id Long,
  // Campi specifici dell'anagrafica
  // Esempi: dataNascita LocalDate, indirizzo String, telefono String, ecc.
}

entity Project {
  projectName String,
  description Text,
  startDate LocalDate,
  endDate LocalDate
}

entity Model {
  modelName String,
  modelType Enumeration {
    Part, Assembly
  },
  filePath String,
  createdAt Instant,
  updatedAt Instant
}

entity Drawing {
  drawingName String,
  filePath String,
  createdAt Instant,
  updatedAt Instant
}

entity FamilyTable {
  instanceName String,
  parameters JsonType
}

entity AssemblyComponent {
  quantity Integer,
  status Enumeration {
    Suppressed, Placed, Fixed
  }
}

entity AttributeType {
  attributeName String,
  attributeDataType Enumeration {
    String, Integer, Real, Boolean
  }
}

entity ParameterLibrary {
  parameterName String
}

entity ModelParameter {
  stringValue String,
  integerValue Integer,
  realValue Double,
  booleanValue Boolean
}

entity DrawingParameter {
  stringValue String,
  integerValue Integer,
  realValue Double,
  booleanValue Boolean
}

entity SheetFormat {
  formatName String,
  width Double,
  height Double
}

entity WorkSheet {
  sheetNumber Integer,
  filePath String,
  createdAt Instant,
  updatedAt Instant
}

entity Layout {
  layoutName String,
  description Text
}

entity LayoutParameter {
  parameterValue String
}

entity LayoutModelAssociation {
  // Associazioni tra layout e modelli
}

entity FileType {
  fileTypeName String
}

entity File {
  fileName String,
  description Text,
  version String,
  filePath String,
  createdAt Instant,
  updatedAt Instant
}

entity Dimension {
  dimensionType String,
  dimensionValue Double,
  unit String
}

entity Symbol {
  symbolName String,
  symbolType String,
  positionX Double,
  positionY Double
}

entity Revision {
  revisionNumber Integer,
  changeDescription Text,
  revisionDate LocalDate
}

// Relazioni tra entità

relationship OneToMany {
  Project to Model,
  Model to Drawing,
  Model to FamilyTable,
  Model to AssemblyComponent,
  AttributeType to ParameterLibrary,
  Model to ModelParameter,
  Drawing to DrawingParameter,
  Drawing to WorkSheet,
  Layout to LayoutParameter,
  Layout to LayoutModelAssociation,
  FileType to File,
  Drawing to Dimension,
  Drawing to Symbol,
  Model to Revision
}

// Relazione estensione User di JHipster con Anagrafic
relationship OneToOne {
  User{userId} to Anagrafic,
  Anagrafic{userId} to User
}

// Autorizzazioni utente
entity Permission {
  accessLevel Enumeration {
    Read, Write, Admin
  }
}

relationship ManyToOne {
  User to Permission,
  Project to Permission
}

// Altre configurazioni
paginate Project, Model with infinite-scroll
```

### Descrizione del File JDL

- **`User`**: Estensione dell'entità utente di JHipster per includere campi come `firstName`, `lastName`, `email` e `password`.
- **`Anagrafic`**: Tabella aggiuntiva per dettagli anagrafici, collegata a `User`.
- **Altre entità**: Sono incluse tutte le tabelle descritte in precedenza, con i rispettivi campi e relazioni definite.
- **Relazioni**: Le relazioni tra le entità sono definite per riflettere la struttura del database, inclusi molti-a-uno, uno-a-molti e uno-a-uno dove necessario.
- **Paginazione**: Configurata la paginazione per `Project` e `Model` utilizzando il metodo di scroll infinito (`infinite-scroll`).
- **Permission**: Gestisce le autorizzazioni degli utenti per i progetti.

Puoi copiare questo file JDL e importarlo nel tuo progetto JHipster utilizzando lo strumento JHipster CLI per generare il codice corrispondente.

Se hai ulteriori domande o modifiche specifiche da apportare, fammelo sapere!