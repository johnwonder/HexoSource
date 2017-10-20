title: angular源码分析之多点嵌入
date: 2017-09-25 09:34:57
tags: angular
---

## angular 多点嵌入举例

先看html
```html
<transclude-multi-slot-point>  
    我是不用指定插入点名称的内容部分  
    <multi-slot-point-tile>我是自定义的标题</multi-slot-point-tile>  
    <multi-slot-point-content>我是自定义的内容</multi-slot-point-content>  
    <multi-slot-point-footer>我是自定义的页脚</multi-slot-point-footer>  
     <multi-slot-point-footer>我是自定义的页脚1</multi-slot-point-footer>
</transclude-multi-slot-point>  
```

再上指令
```js
app.directive("transcludeMultiSlotPoint", function () {  
    var dir = [];  
    dir.replace = false;//看的容易 //不替换也可以
    dir.restrict = "E";  
    dir.transclude = {  
        title: "?multiSlotPointTile",      //注释这行会报 no parent directive的错误   
        content: "?multiSlotPointContent",  
        footer: "?multiSlotPointFooter"  
    };  
    dir.template = "<div>" +  
        "<div>我是模版中的内容</div>" +  //重复属性没用
        "<div ng-transclude='title' ng-transclude='footer'>我是模版中的插入点的标题</div>" +  
        //搞错了 ，ng-transclude不会使hasTranscludeDirective为true 反而因为有compile方法 ，导致会加入postLinkFn 最终执行的事postLinkFn
        "<div ng-transclude='content'>我是模版中的插入点的内容</div>" +  
        "<div class='ng-transclude:footer'>我是模版中的插入点页脚</div>" +  
        "<div ng-transclude>我是模版中的默认插入点的内容</div>" +  
        "</div>";  
    return dir;  
});
```

angular内部首先通过collectDirectives方法会收集到transcludeMultiSlotPoint 指令，然后在applyDirectivesToNode方法中

会把多嵌入点放到当前nodeLinkFn上

### applyDirectivesToNode -> transclude -> slots
```js
if (directiveValue = directive.transclude) {
          hasTranscludeDirective = true;

          //ngif 和ngRepeat 是加了 $$tlb属性的
          // Special case（特殊） ngIf and ngRepeat so that we don't complain about(申诉) duplicate（多个） transclusion.
          // This option should only be used by directives that know how to safely handle element transclusion,
          // where the transcluded nodes are added or replaced after linking.
          if (!directive.$$tlb) {
            assertNoDuplicate('transclusion', nonTlbTranscludeDirective, directive, $compileNode);
            nonTlbTranscludeDirective = directive;
          }

          if (directiveValue == 'element') {
            hasElementTranscludeDirective = true;
            terminalPriority = directive.priority;
            $template = $compileNode;

            //注释节点
            //templateAttrs[directiveName] 在收集指令的时候附的值
            $compileNode = templateAttrs.$$element =
                jqLite(compile.$$createComment(directiveName, templateAttrs[directiveName]));
            compileNode = $compileNode[0];
            replaceWith(jqCollection, sliceArgs($template), compileNode);

            $template[0].$$parentNode = $template[0].parentNode;

            //这里的$template 在transclude element的时候起到了关键作用
            childTranscludeFn = compilationGenerator(mightHaveMultipleTransclusionError, $template, transcludeFn, terminalPriority,
                                        replaceDirective && replaceDirective.name, {
                                          nonTlbTranscludeDirective: nonTlbTranscludeDirective
                                        });
          } else {

            var slots = createMap();

            //compileNode不是jqLite对象 html节点
            $template = jqLite(jqLiteClone(compileNode)).contents();

            if (isObject(directiveValue)) {

              // We have transclusion slots,
              // collect them up, compile them and store their transclusion functions
              $template = [];

              var slotMap = createMap();
              var filledSlots = createMap();

              // Parse the element selectors
              //foreach 为 value  key
              forEach(directiveValue, function(elementSelector, slotName) {
                // If an element selector starts with a ? then it is optional
                var optional = (elementSelector.charAt(0) === '?');
                elementSelector = optional ? elementSelector.substring(1) : elementSelector;

                slotMap[elementSelector] = slotName;

                // We explicitly assign `null` since this implies that a slot was defined but not filled.
                // Later when calling boundTransclusion functions with a slot name we only error if the
                // slot is `undefined`
                slots[slotName] = null;

                // filledSlots contains `true` for all slots that are either optional or have been
                // filled. This is used to check that we have not missed any required slots
                filledSlots[slotName] = optional;
              });

              // Add the matching elements into their slot
              //把符合的元素添加进他们的slot(狭槽)

              //$compileNode 可能包含多个节点
              forEach($compileNode.contents(), function(node) {
                var slotName = slotMap[directiveNormalize(nodeName_(node))];
                //如果映射里有这个slot
                if (slotName) {
                  filledSlots[slotName] = true;
                  slots[slotName] = slots[slotName] || [];
                  slots[slotName].push(node);
                } else {
                  $template.push(node);
                }
              });

              // Check for required slots that were not filled
              //如果optional 为false 且 filledSlotes[slotName] 为false
              forEach(filledSlots, function(filled, slotName) {
                if (!filled) {
                  throw $compileMinErr('reqslot', 'Required transclusion slot `{0}` was not filled.', slotName);
                }
              });

              for (var slotName in slots) {
                if (slots[slotName]) {
                  // Only define a transclusion function if the slot was filled
                  //compilationGenerator返回的是一个函数
                  slots[slotName] = compilationGenerator(mightHaveMultipleTransclusionError, slots[slotName], transcludeFn);
                }
              }
            }

            $compileNode.empty(); // clear contents
            childTranscludeFn = compilationGenerator(mightHaveMultipleTransclusionError, $template, transcludeFn, undefined,
                undefined, { needsNewScope: directive.$$isolateScope || directive.$$newScope});
            childTranscludeFn.$$slots = slots;
          }
}
```

其中节点会把模板的html内容当作子节点添加进去
### nodeLinkFn.transclude = childTranscludeFn
```js
//template 字符串
        if (directive.template) {
          hasTemplate = true;
          assertNoDuplicate('template', templateDirective, directive, $compileNode);
          templateDirective = directive;

          //如果template不是function 那么直接用template的值  
          directiveValue = (isFunction(directive.template))
              ? directive.template($compileNode, templateAttrs)
              : directive.template;

          //判断startSymbol是不是{{
          directiveValue = denormalizeTemplate(directiveValue);

          //在replace情况下 不能存在两个根节点
          //替换掉指令本身自定义节点
          if (directive.replace) {
            replaceDirective = directive;
            if (jqLiteIsTextNode(directiveValue)) {
              $template = [];
            } else {
              $template = removeComments(wrapTemplate(directive.templateNamespace, trim(directiveValue)));
            }
            compileNode = $template[0];

            //如果模板有两个根节点会报错
            if ($template.length != 1 || compileNode.nodeType !== NODE_TYPE_ELEMENT) {
              throw $compileMinErr('tplrt',
                  "Template for directive '{0}' must have exactly one root element. {1}",
                  directiveName, '');
            }

            //替换注释节点
            replaceWith(jqCollection, $compileNode, compileNode);

            var newTemplateAttrs = {$attr: {}};

            //合并原有节点中的指令和模板中的指令

            var templateDirectives = collectDirectives(compileNode, [], newTemplateAttrs);

            //splice() 方法与 slice() 方法的作用是不同的，splice() 方法会直接对数组进行修改
            var unprocessedDirectives = directives.splice(i + 1, directives.length - (i + 1));

            if (newIsolateScopeDirective || newScopeDirective) {
              // The original directive caused the current element to be replaced but this element
              // also needs to have a new scope, so we need to tell the template directives
              // that they would need to get their scope from further up, if they require transclusion

              markDirectiveScope(templateDirectives, newIsolateScopeDirective, newScopeDirective);
            }
            directives = directives.concat(templateDirectives).concat(unprocessedDirectives);
            mergeTemplateAttributes(templateAttrs, newTemplateAttrs);

            ii = directives.length;
          } else {
            //不替换掉的话 如果是 transclude:'element' 那么还是 comment节点

            //不替换掉的话 就把directive.template当作子节点
            //然后在compileNodes  获取childLinkFn -> compileNodes时  也是新的子节点了
            $compileNode.html(directiveValue);
          }
        }

        nodeLinkFn.scope = newScopeDirective && newScopeDirective.scope === true;

        //如果指令没有transclude 或者 transclude为false
        //那么hasTranscludeDirective就为false  line9303
        nodeLinkFn.transcludeOnThisElement = hasTranscludeDirective;//判断是否嵌入
        nodeLinkFn.templateOnThisElement = hasTemplate;//判断是否有模板
        nodeLinkFn.transclude = childTranscludeFn; //$$slots 附加在childTranscludeFn上

        previousCompileContext.hasElementTranscludeDirective = hasElementTranscludeDirective;

        //依赖于是否有templateUrl
        // might be normal or delayed nodeLinkFn depending on if templateUrl is present
        return nodeLinkFn;
```

然后在执行当前节点的compositeLinkFn的时候 去 创建 boundTranscludeFn

然后调用childLinkFn(也就是compositeLinkFn方法)时 传递给 parentBoundTranscludeFn参数

### createBoundTranscludeFn -> boundTranscludeFn
```js
//判断当前节点 指令的 transclude属性是否为true
  if (nodeLinkFn.transcludeOnThisElement) {

              //返回 boundTranscludeFn 方法

              //nodeLinkFn.transclude 就是节点的lazycompliation方法
      childBoundTranscludeFn = createBoundTranscludeFn(
                  scope, nodeLinkFn.transclude, parentBoundTranscludeFn);

   }else if (!nodeLinkFn.templateOnThisElement && parentBoundTranscludeFn) {

      //指令为slot的时候 childBoundTransclude 通常直接为 parentBoundTranscludeFn
      childBoundTranscludeFn = parentBoundTranscludeFn;

  }

   //transcludeFn 有可能是directive的transclude方法

   //调用ngTransclude内置指令compile返回的方法
   //方法里调用controllersTranscludeFn方法
   //再调用boundTranscludeFn
   function createBoundTranscludeFn(scope, transcludeFn, previousBoundTranscludeFn) {
     function boundTranscludeFn(transcludedScope, cloneFn, controllers, futureParentElement, containingScope) {

       if (!transcludedScope) {
         transcludedScope = scope.$new(false, containingScope);
         transcludedScope.$$transcluded = true;
       }
         //调用boundTranscludeFn把 cloneFn传递进来了
         //再调用 publicLinkFn传递
       return transcludeFn(transcludedScope, cloneFn, {
         parentBoundTranscludeFn: previousBoundTranscludeFn,
         transcludeControllers: controllers,
         futureParentElement: futureParentElement
       });
     }

     // We need  to attach the transclusion slots onto the `boundTranscludeFn`
     // so that they are available inside the `controllersBoundTransclude` function
     var boundSlots = boundTranscludeFn.$$slots = createMap();
     for (var slotName in transcludeFn.$$slots) {
       if (transcludeFn.$$slots[slotName]) {
         //transcludeFn.$$slots[slotName] 就是 lazyCompilation
         //把 lazyCompilation放入
         boundSlots[slotName] = createBoundTranscludeFn(scope, transcludeFn.$$slots[slotName], previousBoundTranscludeFn);
       } else {
         boundSlots[slotName] = null;
       }
     }

     return boundTranscludeFn;
   }
```

### nodeLinkFn -> postLinkFn -> controllersBoundTransclude

nodeLinkFn 函数里 会调用postLinkFn 方法，传递controllersBoundTransclude函数

这里的postLinkFn 也就是 ngTransclude内置指令返回的方法

当前节点的模板是在applyDirectivesToNode时通过template属性来调用的

### directive.template
```js
//template 字符串
       if (directive.template) {
         hasTemplate = true;
         assertNoDuplicate('template', templateDirective, directive, $compileNode);
         templateDirective = directive;

         //如果template不是function 那么直接用template的值  
         directiveValue = (isFunction(directive.template))
             ? directive.template($compileNode, templateAttrs)
             : directive.template;

         //判断startSymbol是不是{{
         directiveValue = denormalizeTemplate(directiveValue);

         //在replace情况下 不能存在两个根节点
         //替换掉指令本身自定义节点
         if (directive.replace) {
           replaceDirective = directive;
           if (jqLiteIsTextNode(directiveValue)) {
             $template = [];
           } else {
             $template = removeComments(wrapTemplate(directive.templateNamespace, trim(directiveValue)));
           }
           compileNode = $template[0];

           //如果模板有两个根节点会报错
           if ($template.length != 1 || compileNode.nodeType !== NODE_TYPE_ELEMENT) {
             throw $compileMinErr('tplrt',
                 "Template for directive '{0}' must have exactly one root element. {1}",
                 directiveName, '');
           }

           //替换注释节点
           replaceWith(jqCollection, $compileNode, compileNode);

           var newTemplateAttrs = {$attr: {}};

           //合并原有节点中的指令和模板中的指令
           // combine directives from the original node and from the template:
           // - take the array of directives for this element
           // - split it into two parts, those that already applied (processed) and those that weren't (unprocessed)
           // - collect directives from the template and sort them by priority
           // - combine directives as: processed + template + unprocessed
           var templateDirectives = collectDirectives(compileNode, [], newTemplateAttrs);

           //splice() 方法与 slice() 方法的作用是不同的，splice() 方法会直接对数组进行修改
           var unprocessedDirectives = directives.splice(i + 1, directives.length - (i + 1));

           if (newIsolateScopeDirective || newScopeDirective) {
             // The original directive caused the current element to be replaced but this element
             // also needs to have a new scope, so we need to tell the template directives
             // that they would need to get their scope from further up, if they require transclusion

             markDirectiveScope(templateDirectives, newIsolateScopeDirective, newScopeDirective);
           }
           directives = directives.concat(templateDirectives).concat(unprocessedDirectives);
           mergeTemplateAttributes(templateAttrs, newTemplateAttrs);

           ii = directives.length;
         } else {
           //不替换掉的话 如果是 transclude:'element' 那么还是 comment节点

           //不替换掉的话 就把directive.template当作子节点
           //然后在compileNodes  获取childLinkFn -> compileNodes时  也是新的子节点了
           $compileNode.html(directiveValue);
         }
       }
```

所以ngTransclude内置指令是在编译当前节点的模板子节点时 执行的，我们来看看angular内置的ngTransclude指令

### ngTransclude

```js
var ngTranscludeMinErr = minErr('ngTransclude');
var ngTranscludeDirective = ['$compile', function($compile) {
  return {
    restrict: 'EAC',
    terminal: true,
    compile: function ngTranscludeCompile(tElement) {

      // Remove and cache any original content to act as a fallback
      var fallbackLinkFn = $compile(tElement.contents());
      tElement.empty();

      return function ngTranscludePostLink($scope, $element, $attrs, controller, $transclude) {

        if (!$transclude) {
          throw ngTranscludeMinErr('orphan',
          'Illegal use of ngTransclude directive in the template! ' +
          'No parent directive that requires a transclusion found. ' +
          'Element: {0}',
          startingTag($element));
        }


        // If the attribute is of the form: `ng-transclude="ng-transclude"` then treat it like the default
        if ($attrs.ngTransclude === $attrs.$attr.ngTransclude) {
          $attrs.ngTransclude = '';
        }
        var slotName = $attrs.ngTransclude || $attrs.ngTranscludeSlot;

        // If the slot is required and no transclusion content is provided then this call will throw an error
        //没在指令属性transclude对象中声明插入点属性会报错
        //调用controllersTranscludeFn方法
        $transclude(ngTranscludeCloneAttachFn, null, slotName);

        // If the slot is optional and no transclusion content is provided then use the fallback content
        if (slotName && !$transclude.isSlotFilled(slotName)) {
          useFallbackContent();
        }

        function ngTranscludeCloneAttachFn(clone, transcludedScope) {
          if (clone.length) {
            $element.append(clone);
          } else {
            useFallbackContent();
            // There is nothing linked against the transcluded scope since no content was available,
            // so it should be safe to clean up the generated scope.
            transcludedScope.$destroy();
          }
        }

        function useFallbackContent() {
          // Since this is the fallback content rather than the transcluded content,
          // we link against the scope of this directive rather than the transcluded scope
          fallbackLinkFn($scope, function(clone) {
            $element.append(clone);
          });
        }
      };
    }
  };
}];
```
