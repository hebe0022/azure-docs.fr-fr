---
title: "Routage dépendant des données avec Azure SQL Database | Microsoft Docs"
description: "Utilisation de la classe ShardMapManager dans les applications .NET pour le routage dépendant des données, une fonctionnalité des bases de données partagées dans Azure SQL Database"
services: sql-database
documentationcenter: 
manager: jhubbard
author: torsteng
editor: 
ms.assetid: cad09e15-5561-4448-aa18-b38f54cda004
ms.service: sql-database
ms.custom: scale out apps
ms.workload: Inactive
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/27/2017
ms.author: ddove
ms.openlocfilehash: 2246dd12b922fcbc2e2b58890b3d56253810849c
ms.sourcegitcommit: dfd49613fce4ce917e844d205c85359ff093bb9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/31/2017
---
# <a name="data-dependent-routing"></a>Routage dépendant des données
**Le routage dépendant des données** correspond à la capacité d’utiliser les données dans une requête pour acheminer la demande vers une base de données appropriée. Il s’agit d’un modèle fondamental quand vous travaillez avec des bases de données partitionnées. Le contexte de la demande peut également servir à acheminer la demande, particulièrement si la clé de partitionnement ne fait pas partie de la requête. Chaque requête ou transaction spécifique d’une application utilisant le routage dépendant des données est limitée à l’accès à une seule base de données par demande. Pour les outils élastiques de base de données SQL Azure, ce routage s’effectue avec la classe **[ShardMapManager](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.aspx)** dans les applications ADO.NET.

L’application n’a pas besoin d’effectuer le suivi de diverses chaînes de connexion ou divers emplacements de base de données associés à différents segments de données dans l’environnement partitionné. C’est plutôt le [gestionnaire des cartes de partitions](sql-database-elastic-scale-shard-map-management.md) qui ouvre les connexions aux bases de données appropriées, le cas échéant, en fonction des données de la carte de partitions et de la valeur de la clé de partitionnement qui est la cible de la demande de l’application. Cette clé est généralement l’identificateur *customer_id*, *tenant_id*, *date_key* ou tout autre identificateur spécifique qui est un paramètre fondamental de la demande de la base de données. 

Pour plus d’informations, consultez [Augmentation de la taille des instances SQL Server avec le routage dépendant des données](https://technet.microsoft.com/library/cc966448.aspx).

## <a name="download-the-client-library"></a>Téléchargement de la bibliothèque cliente
Pour obtenir la classe, installez la [bibliothèque cliente de la base de données élastique](http://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/). 

## <a name="using-a-shardmapmanager-in-a-data-dependent-routing-application"></a>Utilisation d’une classe ShardMapManager dans une application de routage dépendant des données
Les applications doivent instancier la classe **ShardMapManager** pendant l’initialisation, en utilisant le répartiteur **[GetSQLShardMapManager](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager.aspx)**. Dans cet exemple, une classe **ShardMapManager** et un objet **ShardMap** spécifique qu’elle contient sont initialisés. Cet exemple illustre les méthodes GetSqlShardMapManager et [GetRangeShardMap](https://msdn.microsoft.com/library/azure/dn824173.aspx) .

    ShardMapManager smm = ShardMapManagerFactory.GetSqlShardMapManager(smmConnnectionString, 
                      ShardMapManagerLoadPolicy.Lazy);
    RangeShardMap<int> customerShardMap = smm.GetRangeShardMap<int>("customerMap"); 

### <a name="use-lowest-privilege-credentials-possible-for-getting-the-shard-map"></a>Utiliser les informations d’identification du niveau de privilège le plus bas possible pour l’obtention de la carte de partitions
Si une application ne manipule pas la carte de partitions proprement dite, les informations d’identification utilisées dans la méthode de fabrique doivent posséder des autorisations d’accès en lecture seule sur la base de données de la **carte de partitions globale** . Ces informations d'identification sont généralement différentes des informations d'identification utilisées pour ouvrir des connexions dans le gestionnaire des cartes de partitions. Consultez aussi [Informations d’identification utilisées pour accéder à la bibliothèque cliente de la base de données élastique](sql-database-elastic-scale-manage-credentials.md). 

## <a name="call-the-openconnectionforkey-method"></a>Appeler la méthode OpenConnectionForKey
La **[méthode ShardMap.OpenConnectionForKey](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkey.aspx)** retourne une connexion ADO.Net prête à émettre des commandes vers la base de données appropriée en fonction de la valeur du paramètre **key**. Les informations de partitions sont mises en cache dans l’application par la classe **ShardMapManager**. De ce fait, ces demandes n’impliquent généralement pas une recherche de base de données par rapport à la base de données **Carte de partitions globale**. 

    // Syntax: 
    public SqlConnection OpenConnectionForKey<TKey>(
        TKey key,
        string connectionString,
        ConnectionOptions options
    )


* Le paramètre **key** est utilisé comme clé de recherche dans la carte de partitions afin de déterminer la base de données appropriée pour la demande. 
* L'instruction **connectionString** est utilisée pour transmettre uniquement les informations d'identification de l'utilisateur correspondant à la connexion de votre choix. Aucun nom de base de données ou de serveur n'est inclus dans l'instruction *connectionString* puisque la méthode détermine la base de données et le serveur à l'aide de l'objet **ShardMap**. 
* **[connectionOptions](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.connectionoptions.aspx)** doit avoir la valeur **ConnectionOptions.Validate** si l’environnement est un environnement où les cartes de partitions peuvent changer et les lignes peuvent se déplacer vers d’autres bases de données suite à des opérations de fractionnement ou de fusion. Cela implique une brève requête vers la carte de partitions locale sur la base de données cible (pas dans la carte de partitions globale) pour que la connexion soit accordée à l’application. 

Si la validation par rapport à la carte de partitions locale échoue (indiquant que le cache est incorrect), le gestionnaire des cartes de partitions interroge la carte de partitions globale afin d'obtenir la nouvelle valeur correcte pour la recherche, de mettre à jour le cache et d'obtenir et retourner la connexion à la base de données appropriée. 

Utilisez **ConnectionOptions.None** uniquement quand aucune modification de mappage de carte n’est prévue pendant qu’une application est en ligne. Dans ce cas, les valeurs en cache peuvent être considérées comme systématiquement correctes, et l'appel de validation aller-retour supplémentaire à la base de données cible peut être ignoré en toute sécurité. Cela réduit le trafic de base de données. L'instruction **connectionOptions** peut également être définie par une valeur d'un fichier de configuration afin d'indiquer si des modifications de partitionnement sont prévues ou non pendant une période donnée.  

Cet exemple utilise la valeur d’une clé entière **CustomerID**, à l’aide d’un objet **ShardMap** nommé **customerShardMap**.  

    int customerId = 12345; 
    int newPersonId = 4321; 

    // Connect to the shard for that customer ID. No need to call a SqlConnection 
    // constructor followed by the Open method.
    using (SqlConnection conn = customerShardMap.OpenConnectionForKey(customerId, 
        Configuration.GetCredentialsConnectionString(), ConnectionOptions.Validate)) 
    { 
        // Execute a simple command. 
        SqlCommand cmd = conn.CreateCommand(); 
        cmd.CommandText = @"UPDATE Sales.Customer 
                            SET PersonID = @newPersonID 
                            WHERE CustomerID = @customerID"; 

        cmd.Parameters.AddWithValue("@customerID", customerId); 
        cmd.Parameters.AddWithValue("@newPersonID", newPersonId); 
        cmd.ExecuteNonQuery(); 
    }  

La méthode **OpenConnectionForKey** retourne une nouvelle connexion déjà ouverte à la base de données. Les connexions utilisées de cette manière tirent pleinement parti du regroupement de connexions ADO.Net. Tant que les transactions et les demandes peuvent être satisfaites une seule partition à la fois, il s'agit de la seule modification nécessaire dans une application utilisant déjà ADO.Net. 

La **[méthode OpenConnectionForKeyAsync](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkeyasync.aspx)** est également disponible si votre application utilise la programmation asynchrone avec ADO.Net. Son comportement est l’équivalent du routage dépendant des données de la méthode **[Connection.OpenAsync](https://msdn.microsoft.com/library/hh223688\(v=vs.110\).aspx)** d’ADO.NET.

## <a name="integrating-with-transient-fault-handling"></a>Intégration de la gestion des erreurs temporaires
Une pratique recommandée dans le développement d’applications d’accès aux données dans le cloud consiste à veiller à ce que les erreurs temporaires soient interceptées par l’application, et à ce que les opérations soient retentées plusieurs fois avant de générer un message d’erreur. La gestion des erreurs temporaires pour les applications cloud est traitée à la page [Gestion des erreurs temporaires](https://msdn.microsoft.com/library/dn440719\(v=pandp.60\).aspx). 

La gestion des erreurs temporaires peut coexister naturellement avec le modèle de routage dépendant des données. La condition clé consiste à réessayer la demande d'accès aux données, notamment le bloc **using** qui a obtenu la connexion de routage dépendant des données. L'exemple précédent pourrait être réécrit comme suit (remarquez la modification mise en surbrillance). 

### <a name="example---data-dependent-routing-with-transient-fault-handling"></a>Exemple : routage dépendant des données avec gestion des erreurs temporaires
<pre><code>int customerId = 12345; 
int newPersonId = 4321; 

<span style="background-color:  #FFFF00">Configuration.SqlRetryPolicy.ExecuteAction(() =&gt; </span> 
<span style="background-color:  #FFFF00">    { </span>
        // Connect to the shard for a customer ID. 
        using (SqlConnection conn = customerShardMap.OpenConnectionForKey(customerId,  
        Configuration.GetCredentialsConnectionString(), ConnectionOptions.Validate)) 
        { 
            // Execute a simple command 
            SqlCommand cmd = conn.CreateCommand(); 

            cmd.CommandText = @&quot;UPDATE Sales.Customer 
                            SET PersonID = @newPersonID 
                            WHERE CustomerID = @customerID&quot;; 

            cmd.Parameters.AddWithValue(&quot;@customerID&quot;, customerId); 
            cmd.Parameters.AddWithValue(&quot;@newPersonID&quot;, newPersonId); 
            cmd.ExecuteNonQuery(); 

            Console.WriteLine(&quot;Update completed&quot;); 
        } 
<span style="background-color:  #FFFF00">    }); </span> 
</code></pre>


Les packages nécessaires pour implémenter la gestion des erreurs temporaires sont automatiquement téléchargés lorsque vous générez l’exemple d’application de base de données élastique. Les packages sont également disponibles séparément dans [Enterprise Library - Bloc des applications de gestion des erreurs temporaires](http://www.nuget.org/packages/EnterpriseLibrary.TransientFaultHandling/). Utilisez la version 6.0 ou ultérieure. 

## <a name="transactional-consistency"></a>Cohérence transactionnelle
Les propriétés transactionnelles sont garanties pour toutes les opérations locales sur une partition. Par exemple, les transactions soumises via le routage dépendant des données s'exécutent dans l'étendue de la partition cible pour la connexion. À ce stade, il n'existe aucune fonctionnalité fournie pour l'inscription de plusieurs connexions dans une transaction, et par conséquent, il n'existe aucune garantie transactionnelle lors des opérations effectuées sur les partitions.

## <a name="next-steps"></a>Étapes suivantes
Pour détacher une partition ou la rattacher, consultez [Utilisation de la classe RecoveryManager pour résoudre les problèmes de carte de partitions](sql-database-elastic-database-recovery-manager.md)

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

