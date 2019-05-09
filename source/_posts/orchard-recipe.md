title: orchard-recipe
date: 2015-09-30 09:07:54
tags: orchard
---

##   Orchard的Recipe是如何解析的

###  SetupService类

安装的时候先去发现Recipe:

```
	_recipeHarvester.HarvestRecipes("Orchard.Setup");
```

###  RecipeHarvester类

```
  /// <summary>
        /// 比如extensionId 为 Orchard.Setup
        /// 找到以.recipe.xml结束的文件
        /// </summary>
        /// <param name="extensionId"></param>
        /// <returns></returns>
        public IEnumerable<Recipe> HarvestRecipes(string extensionId) {
            var recipes = new List<Recipe>();
            var extension = _extensionManager.GetExtension(extensionId);
            if (extension != null) {
		  // ~/Modules/Orchard.Setup/Recipes
                var recipeLocation = Path.Combine(extension.Location, extensionId, "Recipes");
                var recipeFiles = _webSiteFolder.ListFiles(recipeLocation, true);
                recipes.AddRange(
                    from recipeFile in recipeFiles
                    where recipeFile.EndsWith(".recipe.xml", StringComparison.OrdinalIgnoreCase)
                    select _recipeParser.ParseRecipe(_webSiteFolder.ReadFile(recipeFile)));//根据虚拟目录来
            }
            else {
                Logger.Error("Could not discover recipes because module '{0}' was not found.", extensionId);
            }
	    //返回组装好的Recipe
            return recipes;
        }
```

## RecipeParser类

```
public Recipe ParseRecipe(string recipeText) {
            var recipe = new Recipe();

            try {
                var recipeSteps = new List<RecipeStep>();
                TextReader textReader = new StringReader(recipeText);
                var recipeTree = XElement.Load(textReader, LoadOptions.PreserveWhitespace);
                textReader.Close();

                foreach (var element in recipeTree.Elements()) {
                    // Recipe mETaDaTA
                    if (element.Name.LocalName == "Recipe") {
                        foreach (var metadataElement in element.Elements()) {
                            switch (metadataElement.Name.LocalName) {
                                case "Name":
                                    recipe.Name = metadataElement.Value;
                                    break;
                                case "Description":
                                    recipe.Description = metadataElement.Value;
                                    break;
                                case "Author":
                                    recipe.Author = metadataElement.Value;
                                    break;
                                case "WebSite":
                                    recipe.WebSite = metadataElement.Value;
                                    break;
                                case "Version":
                                    recipe.Version = metadataElement.Value;
                                    break;
                                case "Tags":
                                    recipe.Tags = metadataElement.Value;
                                    break;
                                default:
                                    Logger.Error("Unrecognized recipe metadata element {0} encountered. Skipping.", metadataElement.Name.LocalName);
                                    break;
                            }
                        }
                    }
                    // Recipe step
                    else {
                        var recipeStep = new RecipeStep { Name = element.Name.LocalName, Step = element };
                        recipeSteps.Add(recipeStep);
                    }
                }
                recipe.RecipeSteps = recipeSteps;
            }
            catch (Exception exception) {
                Logger.Error(exception, "Parsing recipe failed. Recipe text was: {0}.", recipeText);
                throw;
            }

            return recipe;
        }
```

安装的时候就会通过在界面中选择的model.Recipe去Execute对应的Recipe.
## RecipeManager类

```
    if (recipe == null)
                return null;

            var executionId = Guid.NewGuid().ToString("n");
            _recipeJournal.ExecutionStart(executionId);

            foreach (var recipeStep in recipe.RecipeSteps) {
                _recipeStepQueue.Enqueue(executionId, recipeStep);
            }
            _recipeScheduler.ScheduleWork(executionId);

            return executionId;
```

先把RecipeStep加入recipeStepQueue中，其实内部就是写入一个个文件

## RecipeStepQueue类

```
     public void Enqueue(string executionId, RecipeStep step) {
            var recipeStepElement = new XElement("RecipeStep");
            recipeStepElement.Add(new XElement("Name", step.Name));
            recipeStepElement.Add(step.Step);

            if (_appDataFolder.DirectoryExists(Path.Combine(_recipeQueueFolder, executionId))) {
                int stepIndex = GetLastStepIndex(executionId) + 1;
                _appDataFolder.CreateFile(Path.Combine(_recipeQueueFolder, executionId + Path.DirectorySeparatorChar + stepIndex),
                                          recipeStepElement.ToString());
            }
            else {
                _appDataFolder.CreateFile(
                    Path.Combine(_recipeQueueFolder, executionId + Path.DirectorySeparatorChar + "0"),
                    recipeStepElement.ToString());
            }
        }
```

然后RecipeStepExecutor就去去执行各个Step
## RecipeStepExecutor类

```
public bool ExecuteNextStep(string executionId) {
            var nextRecipeStep= _recipeStepQueue.Dequeue(executionId);
            if (nextRecipeStep == null) {
                _recipeJournal.ExecutionComplete(executionId);
                return false;
            }
            _recipeJournal.WriteJournalEntry(executionId, string.Format("Executing step {0}.", nextRecipeStep.Name));
            var recipeContext = new RecipeContext { RecipeStep = nextRecipeStep, Executed = false };
            try {
                foreach (var recipeHandler in _recipeHandlers) {
                    recipeHandler.ExecuteRecipeStep(recipeContext);
                }
            }
	    ...
```

有没有发现 其实到最后是通过recipeHandler来处理的
我们来看看有多少Handler
FeatureRecipeHandler
CommandRecipeHandler
DataRecipeHandler
MetadataRecipeHandler
MigrationRecipHandler
ModuleRecipeHandler
SettingsRecipeHandler
ThemeRecipeHandler

看看FeatureRecipeHandler内部是怎么ExecuteRecipeStep的

## FeatureRecipeHandler类

```
  // <Feature enable="f1,f2,f3" disable="f4" />
        // Enable/Disable features.
        public void ExecuteRecipeStep(RecipeContext recipeContext) {
            if (!String.Equals(recipeContext.RecipeStep.Name, "Feature", StringComparison.OrdinalIgnoreCase)) {
                return;
            }
```
一开始判断RecipeStep.Name是不是Feature，不是就直接Return.

```
foreach (var featureName in featuresToEnable) {
                if (!availableFeatures.Contains(featureName)) {
                    throw new InvalidOperationException(string.Format("Could not enable feature {0} because it was not found.", featureName));
                }
            }
```

随后去availableFeatures中判断是不是存在featuresToEnable中的某个feature。

```
然后去EnableFeatures.
 if (featuresToEnable.Count != 0) {
                _featureManager.EnableFeatures(featuresToEnable, true);
            }
```


## FeatureManager类

```
  _shellDescriptorManager.UpdateShellDescriptor(shellDescriptor.SerialNumber, enabledFeatures,
                                                              shellDescriptor.Parameters);
```

我们先来看这两段代码：
```
  ShellDescriptor shellDescriptor = _shellDescriptorManager.GetShellDescriptor();
            List<ShellFeature> enabledFeatures = shellDescriptor.Features.ToList();
```
