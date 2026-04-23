[SQLQuery7.sql](https://github.com/user-attachments/files/27010403/SQLQuery7.sql)
# NewRepo-- =============================================
-- SCRIPT CORRIGÉ - DW_Ciments_Bizerte
-- =============================================

USE master;
GO

-- 1. SUPPRIMER ET RECRÉER LA BASE
IF DB_ID('DW_Ciments_Bizerte') IS NOT NULL
BEGIN
    ALTER DATABASE DW_Ciments_Bizerte SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
    DROP DATABASE DW_Ciments_Bizerte;
END
GO

CREATE DATABASE DW_Ciments_Bizerte;
PRINT '✅ Base de données créée';
GO

USE DW_Ciments_Bizerte;
GO

-- 2. CRÉER LES SCHÉMAS
IF NOT EXISTS (SELECT * FROM sys.schemas WHERE name = 'dim')
    EXEC('CREATE SCHEMA dim');

IF NOT EXISTS (SELECT * FROM sys.schemas WHERE name = 'fait')
    EXEC('CREATE SCHEMA fait');
GO

-- =============================================
-- 3. DIMENSIONS
-- =============================================

CREATE TABLE dim.Dim_Date (
    id_date INT PRIMARY KEY,
    date_complete DATE NOT NULL,
    annee INT NOT NULL,
    mois INT NOT NULL,
    jour INT NOT NULL,
    nom_mois VARCHAR(20),
    trimestre TINYINT,
    semestre TINYINT,
    jour_semaine VARCHAR(15),
    est_weekend BIT
);
GO



USE DW_Ciments_Bizerte;
GO

CREATE TABLE dim.Dim_Produit (
    id_produit INT PRIMARY KEY,
    nom_produit VARCHAR(100) NOT NULL,
    categorie_produit VARCHAR(50),
    unite_mesure VARCHAR(20)
);
GO


USE DW_CIMENTS_BIZERTE;
CREATE TABLE dim.Dim_Classe_Ciment (
    id_classe INT PRIMARY KEY,
    designation VARCHAR(50) NOT NULL,
    type_ciment VARCHAR(50),
    resistance VARCHAR(50)
);
GO


CREATE TABLE dim.Dim_Client (
    id_client INT PRIMARY KEY IDENTITY(1,1),
    nom_client VARCHAR(100) NOT NULL
);
GO

CREATE TABLE dim.Dim_Conditionnement (
    id_conditionnement INT PRIMARY KEY,
    type_conditionnement VARCHAR(50) NOT NULL
);
GO

CREATE TABLE dim.Dim_Machine (
    id_machine INT PRIMARY KEY IDENTITY(1,1),
    nom_machine VARCHAR(100) NOT NULL,
    capacite_tonne_heure DECIMAL(10,2)
);
GO


CREATE TABLE dim.Dim_Motif_Arret (
    id_motif_arret INT PRIMARY KEY IDENTITY(1,1),
    type_arret VARCHAR(50) NOT NULL,
    description VARCHAR(255)
);
GO

CREATE TABLE dim.Dim_Article (
    id_article INT PRIMARY KEY,
    code_article VARCHAR(50),
    nom_article VARCHAR(150)
);
GO

CREATE TABLE dim.Dim_Fournisseur (
    id_fournisseur INT PRIMARY KEY IDENTITY(1,1),
    nom_fournisseur VARCHAR(100)
);
GO

CREATE TABLE dim.Dim_Nature_Charge (
    id_charge INT IDENTITY(1,1) PRIMARY KEY,
    nom_charge VARCHAR(100),
    categorie VARCHAR(50),
    type_charge VARCHAR(50)
);
GO


CREATE TABLE dim.Dim_Type_Previsions (
    id_type INT PRIMARY KEY,
    type_prevision VARCHAR(50) NOT NULL
);

-- =============================================
-- 4. TABLES DE FAITS (CORRIGÉES)
-- =============================================

-- 4.1 Fait_Production
CREATE TABLE fait.Fait_Production (
    id_date INT NOT NULL,                      -- ✅ PK + FK → NOT NULL
    id_produit INT NOT NULL,                   -- ✅ PK + FK → NOT NULL
    id_machine INT NOT NULL,                   -- ✅ PK + FK → NOT NULL meme four pour chaux
    quantite_produite DECIMAL(18,3) NOT NULL,  -- ✅ Mesure → NOT NULL
    heure_marche DECIMAL(10,2) NULL,           -- ✅ Mesure → NULL (Chaux n'a pas cette donnée)

    CONSTRAINT PK_Fait_Production 
        PRIMARY KEY (id_date, id_produit, id_machine),  -- ✅ id_classe dans la PK

    CONSTRAINT FK_prod_date 
        FOREIGN KEY (id_date) REFERENCES dim.Dim_Date(id_date),

    CONSTRAINT FK_prod_produit 
        FOREIGN KEY (id_produit) REFERENCES dim.Dim_Produit(id_produit),

    CONSTRAINT FK_prod_machine 
        FOREIGN KEY (id_machine) REFERENCES dim.Dim_Machine(id_machine)
);
GO

-- 4.2 Fait_Vente_Commerciale
CREATE TABLE fait.Fait_Vente_Commerciale (
    id_date INT NOT NULL,                      -- ✅ PK + FK → NOT NULL
    id_produit INT NOT NULL,                   -- ✅ PK + FK → NOT NULL
    id_conditionnement INT NOT NULL,           -- ✅ PK + FK → NOT NULL
    id_classe INT NOT NULL DEFAULT 0,          -- ✅ PK + FK → NOT NULL + DEFAULT 0
    quantite_vendue DECIMAL(18,3) NOT NULL,    -- ✅ Mesure → NOT NULL

    CONSTRAINT PK_FVC PRIMARY KEY (id_date, id_produit, id_conditionnement, id_classe),

    CONSTRAINT FK_vc_date FOREIGN KEY (id_date)
        REFERENCES dim.Dim_Date(id_date),

    CONSTRAINT FK_vc_produit FOREIGN KEY (id_produit)
        REFERENCES dim.Dim_Produit(id_produit),

    CONSTRAINT FK_vc_cond FOREIGN KEY (id_conditionnement)
        REFERENCES dim.Dim_Conditionnement(id_conditionnement),

    CONSTRAINT FK_vc_classe FOREIGN KEY (id_classe)
        REFERENCES dim.Dim_Classe_Ciment(id_classe)
);
GO

-- 4.3 Fait_Vente_Logistique
CREATE TABLE fait.Fait_Vente_Logistique (
    id_date INT NOT NULL,                      -- ✅ PK + FK → NOT NULL
    id_produit INT NOT NULL,                   -- ✅ PK + FK → NOT NULL
    id_client INT NOT NULL,                    -- ✅ PK + FK → NOT NULL
    id_conditionnement INT NOT NULL,           -- ✅ PK + FK → NOT NULL
    quantite_vendue DECIMAL(18,3) NOT NULL,    -- ✅ Mesure → NOT NULL
    prix_unitaire DECIMAL(18,3) NULL,          -- ✅ Mesure → NULL (optionnel)
    valeur_ht DECIMAL(18,3) NULL,              -- ✅ Mesure → NULL (optionnel)

    CONSTRAINT PK_FVL PRIMARY KEY (id_date, id_produit, id_client, id_conditionnement),

    CONSTRAINT FK_vl_date FOREIGN KEY (id_date)
        REFERENCES dim.Dim_Date(id_date),

    CONSTRAINT FK_vl_produit FOREIGN KEY (id_produit)
        REFERENCES dim.Dim_Produit(id_produit),

    CONSTRAINT FK_vl_client FOREIGN KEY (id_client)
        REFERENCES dim.Dim_Client(id_client),

    CONSTRAINT FK_vl_cond FOREIGN KEY (id_conditionnement)
        REFERENCES dim.Dim_Conditionnement(id_conditionnement)
);
GO

-- 4.4 Fait_Achat
CREATE TABLE fait.Fait_Achat (
    id_date INT NOT NULL,                      -- ✅ PK + FK → NOT NULL
    id_article INT NOT NULL,                   -- ✅ PK + FK → NOT NULL
    id_fournisseur INT NOT NULL,               -- ✅ PK + FK → NOT NULL
    quantite DECIMAL(18,3) NOT NULL,           -- ✅ Mesure → NOT NULL
    prix_unitaire DECIMAL(18,4) NULL,          -- ✅ Mesure → NULL
    montant DECIMAL(18,3) NULL,                -- ✅ Mesure → NULL

    CONSTRAINT PK_FA PRIMARY KEY (id_date, id_article, id_fournisseur),

    CONSTRAINT FK_achat_date FOREIGN KEY (id_date)
        REFERENCES dim.Dim_Date(id_date),

    CONSTRAINT FK_achat_article FOREIGN KEY (id_article)
        REFERENCES dim.Dim_Article(id_article),

    CONSTRAINT FK_achat_fournisseur FOREIGN KEY (id_fournisseur)
        REFERENCES dim.Dim_Fournisseur(id_fournisseur)
);
GO

-- 4.5 Fait_Arret_Machine
CREATE TABLE fait.Fait_Arret_Machine (
    id_date INT NOT NULL,                      -- ✅ PK + FK → NOT NULL
    id_machine INT NOT NULL,                   -- ✅ PK + FK → NOT NULL
    id_motif_arret INT NOT NULL,               -- ✅ PK + FK → NOT NULL
    duree_arret INT NOT NULL,                  -- ✅ Mesure → NOT NULL
    heure_marche DECIMAL(10,2) NULL,           -- ✅ Mesure → NULL

    CONSTRAINT PK_FAM PRIMARY KEY (id_date, id_machine, id_motif_arret),

    CONSTRAINT FK_arret_date FOREIGN KEY (id_date)
        REFERENCES dim.Dim_Date(id_date),

    CONSTRAINT FK_arret_machine FOREIGN KEY (id_machine)
        REFERENCES dim.Dim_Machine(id_machine),

    CONSTRAINT FK_arret_motif FOREIGN KEY (id_motif_arret)
        REFERENCES dim.Dim_Motif_Arret(id_motif_arret)
);
GO

-- 4.6 Fait_Cout_Production
CREATE TABLE fait.Fait_Cout_Production (
    id_date INT NOT NULL,                      -- ✅ PK + FK → NOT NULL
    id_produit INT NOT NULL,                   -- ✅ PK + FK → NOT NULL
    id_classe INT NOT NULL,          -- ✅ PK + FK → NOT NULL + DEFAULT 0
    id_charge INT NOT NULL,                    -- ✅ PK + FK → NOT NULL
    cout_absolu DECIMAL(18,2) NOT NULL,        -- ✅ Mesure → NOT NULL
              
    CONSTRAINT PK_FCP PRIMARY KEY (id_date, id_produit, id_classe, id_charge),

    CONSTRAINT FK_cout_date FOREIGN KEY (id_date)
        REFERENCES dim.Dim_Date(id_date),

    CONSTRAINT FK_cout_produit FOREIGN KEY (id_produit)
        REFERENCES dim.Dim_Produit(id_produit),

    CONSTRAINT FK_cout_classe FOREIGN KEY (id_classe)
        REFERENCES dim.Dim_Classe_Ciment(id_classe),

    CONSTRAINT FK_cout_charge FOREIGN KEY (id_charge)
        REFERENCES dim.Dim_Nature_Charge(id_charge)
);
GO

-- 4.7 Fait_Previsions
CREATE TABLE fait.Fait_Previsions (
    id_date INT NOT NULL,                      -- ✅ PK + FK → NOT NULL
    id_produit INT NOT NULL,                   -- ✅ PK + FK → NOT NULL
    id_classe INT NOT NULL DEFAULT 0,          -- ✅ PK + FK → NOT NULL + DEFAULT 0
    id_type INT NOT NULL,                      -- ✅ PK + FK → NOT NULL
    quantite_prevu DECIMAL(18,3) NOT NULL,     -- ✅ Mesure → NOT NULL
    ca_objectif DECIMAL(18,2) NULL,            -- ✅ Mesure → NULL

    CONSTRAINT PK_FP PRIMARY KEY (id_date, id_produit, id_classe, id_type),

    CONSTRAINT FK_prev_date FOREIGN KEY (id_date)
        REFERENCES dim.Dim_Date(id_date),

    CONSTRAINT FK_prev_produit FOREIGN KEY (id_produit)
        REFERENCES dim.Dim_Produit(id_produit),

    CONSTRAINT FK_prev_classe FOREIGN KEY (id_classe)
        REFERENCES dim.Dim_Classe_Ciment(id_classe),

    CONSTRAINT FK_prev_type FOREIGN KEY (id_type)
        REFERENCES dim.Dim_Type_Previsions(id_type)
);
GO

-- 4.8 Fait_Suivi_Appro
CREATE TABLE fait.Fait_Suivi_Appro (
    id_date INT NOT NULL,                      -- ✅ PK + FK → NOT NULL
    nb_requetes INT NOT NULL,                  -- ✅ Mesure → NOT NULL
    nb_commandes INT NOT NULL,                 -- ✅ Mesure → NOT NULL

    CONSTRAINT PK_FSA PRIMARY KEY (id_date),

    CONSTRAINT FK_appro_date FOREIGN KEY (id_date)
        REFERENCES dim.Dim_Date(id_date)
);
GO

-- =============================================
-- 5. VÉRIFICATION
-- =============================================

PRINT '';
PRINT '===========================================';
PRINT '✅ CRÉATION DU DWH TERMINÉE AVEC SUCCÈS !';
PRINT '===========================================';
PRINT '';

SELECT 
    s.name AS Schema_Name,
    t.name AS Table_Name
FROM sys.tables t
JOIN sys.schemas s ON t.schema_id = s.schema_id
WHERE s.name IN ('dim', 'fait')
ORDER BY s.name, t.name;
GO




USE DW_Ciments_Bizerte;
GO

-- =============================================
-- 1. DIM_DATE (Générer 2019-2022)
-- =============================================

DECLARE @start_date DATE = '2019-01-01';
DECLARE @end_date DATE = '2022-12-31';
DECLARE @current_date DATE = @start_date;

WHILE @current_date <= @end_date
BEGIN
    INSERT INTO dim.Dim_Date VALUES (
        YEAR(@current_date) * 10000 + MONTH(@current_date) * 100 + DAY(@current_date), -- id_date
        @current_date,                                                                 -- date_complete
        YEAR(@current_date),                                                           -- annee
        MONTH(@current_date),                                                          -- mois
        DAY(@current_date),                                                            -- jour
        DATENAME(MONTH, @current_date),                                                -- nom_mois
        (MONTH(@current_date) - 1) / 3 + 1,                                            -- trimestre
        CASE WHEN MONTH(@current_date) <= 6 THEN 1 ELSE 2 END,                         -- semestre
        DATENAME(WEEKDAY, @current_date),                                              -- jour_semaine
        CASE WHEN DATEPART(WEEKDAY, @current_date) IN (1, 7) THEN 1 ELSE 0 END         -- est_weekend
    );
    SET @current_date = DATEADD(DAY, 1, @current_date);
END

PRINT '✅ Dim_Date chargée (' 
    + CAST(DATEDIFF(DAY, @start_date, @end_date) + 1 AS VARCHAR) 
    + ' jours de 2019 à 2022)';
GO


USE DW_CIMENTS_BIZERTE;
INSERT INTO dim.Dim_Produit VALUES 
(100, 'Ciment', 'Produit Fini', 'Tonne'),
(101, 'Chaux Hydraulique', 'Produit Fini', 'Tonne'),
(102, 'Calcaire', 'Matiere Premiere', 'Tonne'),
(103, 'Clinker', 'Matiere Semi_Fini', 'Tonne'),
(104, 'Coke', 'Combustible', 'Tonne'),
(105, 'Poudre Cru', 'Matiere Intermediaire', 'Tonne');

PRINT '✅ Dim_Produit chargée (6 lignes)';
GO   


INSERT INTO dim.Dim_Classe_Ciment VALUES 
(0, 'NA', 'Non Applicable', '-'),
(1, 'CEMIIA-L32,5N', 'Composé', 'Moyenne'),
(2, 'CEMI42,5N', 'Pur', 'Solide');

PRINT '✅ Dim_Classe_Ciment chargée (3 lignes)';
GO

INSERT INTO dim.Dim_Conditionnement VALUES 
(1, 'Sac'),
(2, 'Vrac');

PRINT '✅ Dim_Conditionnement chargée (2 lignes)';
GO

INSERT INTO dim.Dim_Machine (nom_machine) VALUES 
('Concasseur Carrière'),
('Broyeur Coke'),
('Broyeur Cru'),
('Four Rotatif'),
('Broyeur Ciment');

PRINT '✅ Dim_Machine chargée (5 lignes)';
GO


INSERT INTO dim.Dim_Type_Previsions VALUES 
(1, 'Vente'),
(2, 'Production');

PRINT '✅ Dim_Type_Previsions chargée (3 lignes)';
GO




INSERT INTO dim.Dim_Nature_Charge (nom_charge, categorie, type_charge) VALUES
-- Charges directes
('Clinker (Direct)', 'Matiere Premiere', 'Direct'),
('Gypse', 'Matiere Premiere', 'Direct'),
('Main d''œuvre (Direct)', 'Personnel', 'Direct'),
('Charges sociales (Direct)', 'Personnel', 'Direct'),
('Electricité (Direct)', 'Energie', 'Direct'),
('Eau', 'Energie', 'Direct'),

-- Charges indirectes
('Clinker (Indirect)', 'Matiere Premiere', 'Indirect'),
('Main d''œuvre (Indirect)', 'Personnel', 'Indirect'),
('Charges sociales (Indirect)', 'Personnel', 'Indirect'),
('Fournitures d''entretien', 'Maintenance', 'Indirect'),
('Amortissement Gros entretien', 'Financier', 'Indirect'),

-- Charges de structure
('Achats (60)', 'Structure', 'Structure'),
('Services extérieurs (61)', 'Structure', 'Structure'),
('Autres services extérieurs (62)', 'Structure', 'Structure'),
('Charges diverses (63)', 'Structure', 'Structure'),
('Charges de Personnel (64)', 'Personnel', 'Structure'),
('Charges financières (65)', 'Financier', 'Structure'),
('Impôts et taxes (66)', 'Fiscalité', 'Structure'),
('Amortissements (68)', 'Financier', 'Structure'),

-- Charges commerciales
('Electricité (Commercial)', 'Energie', 'Commercial'),
('Main d''œuvre (Commercial)', 'Personnel', 'Commercial'),
('Charges sociales (Commercial)', 'Personnel', 'Commercial'),
('Fournitures d''entretien (Commercial)', 'Maintenance', 'Commercial'),
('Amortissement Gros entretien (Commercial)', 'Financier', 'Commercial'),
('Contribution transport', 'Commercial', 'Commercial'),
('Sacherie', 'Commercial', 'Commercial');
GO

PRINT '✅ Dim_Nature_Charge chargée (24 lignes)';




USE DW_Ciments_Bizerte;
GO

-- Vider la table si elle existe déjà
--UNCATE TABLE dim.Dim_Motif_Arret;

-- Insérer les 45 motifs standardisés
INSERT INTO dim.Dim_Motif_Arret (type_arret, description) VALUES
-- MÉCANIQUE (1-4)
('Mécanique', 'Coincement/Bourrage équipement'),
('Mécanique', 'Défaut rotation/Signal retour'),
('Mécanique', 'Intervention MECA équipement'),
('Mécanique', 'Usure/Détérioration matériel'),

-- ÉLECTRIQUE (5-8)
('Électrique', 'Défaut Watch-Dog'),
('Électrique', 'Problème moteur/Démarreur'),
('Électrique', 'Problème alimentation électrique'),
('Électrique', 'Intervention ELEC/ERI'),

-- PROCESS (9-12)
('Process', 'Bouchage équipement'),
('Process', 'Défaut évacuation filtre'),
('Process', 'Problème séparateur/Vis'),
('Process', 'Réduction vitesse four'),

-- ALIMENTATION (13-15)
('Alimentation', 'Manque alimentation matière'),
('Alimentation', 'Manque combustible'),
('Alimentation', 'Problème doseur/alimentateur'),

-- TEMPÉRATURE (16-19)
('Température', 'Température palier entrée'),
('Température', 'Température palier séparateur'),
('Température', 'Surchauffe équipement'),
('Température', 'Température huile/réducteur'),

-- SÉCURITÉ (20-23)
('Sécurité', 'Défaut surcharge'),
('Sécurité', 'Défaut position clapet'),
('Sécurité', 'Défaut pression air'),
('Sécurité', 'Défaut niveau silo'),

-- ENTRETIEN (24-27)
('Entretien', 'Arrêt programmé'),
('Entretien', 'Entretien général'),
('Entretien', 'Visite inspection'),
('Entretien', 'Intervention presse-étoupe'),

-- DÉMARRAGE (28-30)
('Démarrage', 'Chauffage équipement'),
('Démarrage', 'Démarrage équipement'),
('Démarrage', 'Remplissage trémies'),

-- ARRÊT CASCADE (31-33)
('Arrêt Cascade', 'Arrêt suite arrêt four'),
('Arrêt Cascade', 'Arrêt suite arrêt broyeur'),
('Arrêt Cascade', 'Arrêt ventilateur refroidisseur'),

-- ARRÊT VOLONTAIRE (34-35)
('Arrêt Volontaire', 'Arrêt volontaire silo plein'),
('Arrêt Volontaire', 'Arrêt volontaire intervention'),

-- DÉFAUT SPÉCIFIQUE (36-42)
('Défaut Spécifique', 'Défaut HAZLER'),
('Défaut Spécifique', 'Défaut SAS alimentation'),
('Défaut Spécifique', 'Défaut bande transporteuse'),
('Défaut Spécifique', 'Défaut élévateur'),
('Défaut Spécifique', 'Défaut pompe'),
('Défaut Spécifique', 'Défaut grille/refroidisseur'),
('Défaut Spécifique', 'Défaut concasseur'),

-- DIVERS (43-45)
('Divers', 'RAS (Sans arrêt)'),
('Divers', 'Problème indéterminé'),
('Divers', 'Autres motifs');

PRINT '✅ Dim_Motif_Arret chargée (45 motifs standardisés)';
GO

-- Vérification
SELECT type_arret, COUNT(*) as nb_motifs
FROM dim.Dim_Motif_Arret
GROUP BY type_arret
ORDER BY type_arret;
GO

USE DW_Ciments_Bizerte;
GO
SELECT TOP 200*
FROM dim.Dim_Motif_Arret;

USE DW_Ciments_Bizerte;
GO
SELECT TOP 200*
FROM fait.Fait_Arret_Machine;
---JOIN dim.Dim_Date dd ON fp.id_date = dd.id_date
---WHERE dd.annee = 2020
---ORDER BY fp.id_date;




USE DW_Ciments_Bizerte;
GO
SELECT TOP 200*
FROM fait.Fait_Production;


SELECT * FROM dim.Dim_Date;
SELECT * FROM dim.Dim_Produit;
SELECT * FROM dim.Dim_Classe_Ciment;
SELECT * FROM dim.Dim_Conditionnement;
SELECT * FROM dim.Dim_Machine;
SELECT * FROM dim.Dim_Nature_Charge;
SELECT * FROM dim.Dim_Type_Previsions;      
