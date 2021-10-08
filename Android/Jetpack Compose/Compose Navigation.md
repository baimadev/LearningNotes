## NavController

```kotlin
      val navController = rememberNavController()
      //当前回退栈
      val backStackEntry = navController.currentBackStackEntryAsState()
      var currentScreen = RallyScreen.fromRoute(backStackEntry.value?.destination?.route)

      //导航
      navController.navigate(screen.name)

        NavHost(
                      navController = navController,
                      startDestination = RallyScreen.Overview.name,
                      modifier = Modifier.padding(innerPadding)
                  )  {

                      composable(RallyScreen.Overview.name) {
                          OverviewBody(onClickSeeAllAccounts = {
                              navController.navigate(RallyScreen.Accounts.name)
                          },
                              onClickSeeAllBills = { navController.navigate(RallyScreen.Bills.name) }
                          )
                      }

                      composable(route = RallyScreen.Accounts.name){
                          AccountsBody(accounts = UserData.accounts){ name ->
                              navigateToSingleAccount(navController,name)
                          }
                      }

                      composable(RallyScreen.Bills.name){
                          BillsBody(bills = UserData.bills)
                      }
                  }

```

## 传参

```kotlin
NavHost(...) {
    ...
    composable(
        "$accountsName/{name}",
        arguments = listOf(
            navArgument("name") {
            // Make argument type safe
            type = NavType.StringType
            }
        )
    ) { entry -> // Look up "name" in NavBackStackEntry's arguments
        val accountName = entry.arguments?.getString("name")
   }
}

```