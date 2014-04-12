## Interact with relational databases in Haskell [![Build Status](https://travis-ci.org/lykahb/groundhog.png?branch=master)](https://travis-ci.org/lykahb/groundhog)

Migrate and access PostgreSQL, MySQL, and SQLite with type safety and detailed control. 
Advanced migration capabilities allow you to precisely specify the schema description, fitting it to an existing database, or
creating a migration script for a new one. Groundhog is not opinionated about schema and can bind your datatypes to a relational model which may have composite keys, references across different schemas, indexes, etc.
It is used in hobby projects and commercial applications.

## Useful links

* [Tutorial](http://www.fpcomplete.com/user/lykahb/groundhog).
* Full list of [examples](examples).
* Read [docs](http://hackage.haskell.org/package/groundhog) for
groundhog on Hackage.
* Read [docs](http://hackage.haskell.org/package/groundhog-th/docs/Database-Groundhog-TH.html) for the
`mkPersist` mapping description on Hackage.

### Creating and migrating tables

Here is a simple example of a database of Machines and Parts where
Machines have many Parts. It creates the tables and links them together.

```haskell
{-# LANGUAGE GADTs, TypeFamilies, TemplateHaskell, QuasiQuotes, FlexibleInstances, StandaloneDeriving #-}
import Control.Monad.IO.Class (liftIO)
import Database.Groundhog.TH
import Database.Groundhog.Sqlite

data Machine = Machine { modelName :: String, cost :: Double } deriving Show
data Part = Part { partName :: String, weight :: Int, machine :: DefaultKey Machine }
deriving instance Show Part

mkPersist defaultCodegenConfig [groundhog|
- entity: Machine
- entity: Part
|]

main = withSqliteConn ":memory:" $ runDbConn $ do
  runMigration defaultMigrationLogger $ do
    migrate (undefined :: Machine)
    migrate (undefined :: Part)
```

### Inserting values

```haskell
megatron <- insert $ Machine "Megatron 5000" 2500.00
insert $ Part "Megamaker" 50 megatron
insert $ Part "Tiny Bolt" 1 megatron

microtron <- insert $ Machine "Microtron 12" 19.99
insert $ Part "Insignificonium" 2 microtron
```

### Querying results

```haskell
megatronFromDB <- get megatron
liftIO $ putStrLn $ "Megatron from DB: " ++ show megatronFromDB
  
parts <- select $ (MachineField ==. megatron &&. lower PartNameField `notLike` "%tiny%") `orderBy` [Asc PartNameField]
liftIO $ putStrLn $ "Big parts for the Megatron: " ++ show parts
```
