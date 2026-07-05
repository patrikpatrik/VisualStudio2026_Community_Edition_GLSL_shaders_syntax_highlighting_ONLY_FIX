#  A potential fix/workaround for Visual Studio 2026 Community Edition with enabling GLSL shaders with syntax highlighting ONLY.
You do not have to download anything from the marketplace. This is done without.
**If TL;DR, skip to bottom paragraph!**

This is for anyone who had frustrating problems with having multi-staged shaders into a *single shader file* such as:
  - vs -> fs
  - vs -> gs -> fs
  - vs -> tcs -> tes -> fs
  - vs -> tcs -> tes -> gs -> fs
  - cs

# The problem is [shaderlab.json](shaderlab.json)
For Visual Studio 2026 it is located here:
`C:\Program Files\Microsoft Visual Studio\18\Community\Common7\IDE\CommonExtensions\Microsoft\TextMate\Starterkit\Extensions\shaderlab\syntaxes\shaderlab.json`

You may have noticed this from back as early as **Visual Studio 2019**, possibly even **Visual Studio 2017**. The first extensionless file called ***shader*** will 
show syntax highlighting, yet for 'shader2', 'shader3', 'shader4', etc.. will not.

Partial solution to this problem was: Tools -> Options -> Text Editor -> File Extensions -> adding your extension `.glsl` and selecting
the drop down menu such as Visual Studio C++. You would get the syntax highlighting *plus* error correction. However, this error correction becomes 
problematic because of certain parsing tokens to combine multi-staged shaders into one shader file which need more workarounds. For instance:
```
#vertex shader
#version 460 core
... code here ...

#fragment shader
#version 460 core
```
These would show up as error squiggly lines on line 2 `#vertex shader` & line 3 `#version 460 core`. You could add `//` marks before them
but then you would have to keep track of all your shader files and so forth and at this point its just another fix of a fix.

This solution allows syntax highlighting ***only***, no error validation.

Open shaderlab.json with any text editor and you will see why the extensionless `shader` file works.
<img width="1290" height="225" alt="shaderFile" src="https://github.com/user-attachments/assets/a1ffc226-a4e5-4341-a6dd-6adb9d1a89ee" />

This is the answer. Simply add any file what you want, and Visual Studio will now default the context highlighting to any GLSL shader file you put here.
<img width="978" height="511" alt="multiples" src="https://github.com/user-attachments/assets/e2eaa340-da13-4fa2-930f-c641a0248187" />

If you want to stop here, save the file and restart Visual Studio 2026. Remove all your shader files & reload them again. You now have your GLSL code
being highlighted without any validation errors.

If you took a closer look at the example shader file with the GLSL context highlighting, you'll notice that not all keywords highlighted such as `gl_Position,
_MainTex, COLOR, Albedo, worldPos etc..` There was a bug in the matching conditional statement with certain keywords that silently errored out. This is what existed
for possibly 10+ years in Visual Studio IDE.

To highlight all GLSL keywords to shaderlab.json? Copy the rest of this portion of code which allows all OpenGL 4.6 keywords to highlight as well.
```
"patterns": [
    {
      "match": "\\b([0-9]+\\.?[0-9]*)\\b",
      "name": "constant.numeric.shaderlab"
    },
    {
      "match": "\\b(Shader|Properties|Category|SubShader|Tags|Pass)\\b",
      "name": "support.class.shaderlab"
    },
    {
      "match": "\\b(CGPROGRAM|ENDCG)\\b",
      "name": "support.class.shaderlab"
    },
    {
      "match": "\\b(Dependency|Fallback)\\b",
      "name": "support.variable.shaderlab"
    },
    {
      "match": "^\\s*\\[(HideInInspector)\\]",
      "name": "keyword.shaderlab"
    },
    {
      "include": "#comments"
    },
    {
      "include": "#strings"
    },
    {
      "include": "#functions"
    },
    {
      "include": "#cg"
    }
  ],
  "repository": {
    "comments": {
      "patterns": [
        {
          "begin": "//",
          "end": "$",
          "name": "comment.line.double-slash.shaderlab"
        },
        {
          "begin": "/\\*",
          "end": "\\*/",
          "name": "comment.line.block.shaderlab"
        }
      ]
    },
    "strings": {
      "patterns": [
        {
          "begin": "\"",
          "end": "\"",
          "name": "string.quoted.double.shaderlab"
        }
      ]
    },
    "functions": {
      "patterns": [
        {
          "match": "(?x) (?: (?= \\s )  (?:(?<=else|new|return) | (?<!\\w)) (\\s+))?\n\t\t\t(\\b \n\t\t\t\t(?!(while|for|do|if|else|switch|catch|enumerate|return|sizeof|[cr]?iterate)\\s*\\()(?:(?!NS)[A-Za-z_][A-Za-z0-9_]*+\\b | :: )++\t\t\t\t  # actual name\n\t\t\t)\n\t\t\t \\s*(\\()",
          "captures": {
            "1": {
              "name": "punctuation.whitespace.function-call.leading.shaderlab"
            },
            "2": {
              "name": "support.function.any-method.shaderlab"
            },
            "3": {
              "name": "punctuation.definition.parameters.shaderlab"
            }
          },
          "name": "meta.function-call.shaderlab"
        }
      ]
    },
    "cg": {
      "patterns": [
        {
          "match": "^\\s*#\\s*(define|defined|elif|else|if|ifdef|ifndef|line|pragma|undef|version|extension|include|error)\\b",
          "name": "entity.name.class.shaderlab"
        },
        {
          "match": "\\.([rgba]{1,4}|[xyzw]{1,4}|[stpq]{1,4})\\b",
          "name": "entity.name.class.shaderlab"
        },
        {
          "match": "\\b(if|else|for|while|do|return|break|continue|switch|case|default|discard)\\b",
          "name": "string.shaderlab"
        },
        {
          "match": "\\b(const|extern|in|inline|inout|static|out|uniform|attribute|varying|layout|invariant|precision|highp|mediump|lowp|flat|smooth|noperspective|centroid|sample|patch|readonly|writeonly|coherent|volatile|restrict|shared|buffer|subroutine)\\b",
          "name": "string.shaderlab"
        },
        {
          "match": "\\b(gl_Position|gl_PointSize|gl_ClipDistance|gl_CullDistance|gl_FragCoord|gl_FrontFacing|gl_FragDepth|gl_PointCoord|gl_PrimitiveID|gl_SampleID|gl_SamplePosition|gl_SampleMaskIn|gl_SampleMask|gl_Layer|gl_ViewportIndex|gl_VertexID|gl_InstanceID|gl_InvocationID|gl_FragColor|gl_FragData|gl_GlobalInvocationID|gl_LocalInvocationID|gl_LocalInvocationIndex|gl_WorkGroupSize|gl_WorkGroupID|gl_NumWorkGroups|gl_PrimitiveIDIn|gl_PrimitiveIndicesNV|gl_TessCoord|gl_TessLevelOuter|gl_TessLevelInner|gl_PatchVerticesIn|gl_MaxVertexAttribs|gl_MaxCombinedTextureImageUnits|gl_MaxDrawBuffers|gl_MaxFragmentUniformComponents|gl_MaxTextureImageUnits|gl_MaxVaryingFloats|gl_MaxVertexUniformComponents|gl_MaxVertexTextureImageUnits|gl_DepthRange|gl_BaseVertex|gl_BaseInstance|gl_DrawID|gl_HelperInvocation|gl_SubgroupID|gl_SubgroupSize|gl_SubgroupInvocation|gl_BoundingBox|gl_ViewportMask|gl_MaxSamples|gl_MaxImageUnits|gl_MaxComputeWorkGroupCount|gl_MaxComputeWorkGroupSize|gl_MaxGeometryOutputVertices|gl_MaxPatchVertices|gl_MinProgramTexelOffset|gl_MaxProgramTexelOffset|gl_MaxClipDistances|gl_MaxCullDistances|gl_DepthRange|_MainTex|_Color|_Time|_SinTime|_CosTime|unity_DeltaTime|_ProjectionParams|_ScreenParams|viewDir|COLOR|screenPos|worldPos|worldRefl|worldNormal|Albedo|Normal|Emission|Specular|Gloss|Alpha|_Object2World|_World2Object|_WorldSpaceCameraPos|unity_Scale|_ObjectSpaceLightPos|appdata_base|appdata_tan|appdata_full|appdata_img|SurfaceOutput|UNITY_MATRIX_MVP|UNITY_MATRIX_MV|UNITY_MATRIX_V|UNITY_MATRIX_P|UNITY_MATRIX_VP|UNITY_MATRIX_T_MV|UNITY_MATRIX_IT_MV|UNITY_MATRIX_TEXTURE0|UNITY_MATRIX_TEXTURE1|UNITY_MATRIX_TEXTURE2|UNITY_MATRIX_TEXTURE3|void|struct|typedef|signed|unsigned|double|dvec[234]|dmat[234]|dmat[234]x[234]|float|vec[234]|mat[234]|mat[234]x[234]|half|fixed|long|int|ivec[234]|short|char|bool|bvec[234]|uint|uvec[234]|sampler[12]D|sampler3D|samplerCube|sampler[12]DShadow|samplerCubeShadow|sampler[12]DArray|sampler[12]DArrayShadow|sampler2DRect|sampler2DRectShadow|samplerBuffer|sampler2DMS|sampler2DMSArray|samplerCubeArray|samplerCubeArrayShadow|sampler1D|sampler1DShadow|sampler1DArray|sampler1DArrayShadow|isampler[123]D|isamplerCube|isampler[12]DArray|isampler2DRect|isampler2DMS|isampler2DMSArray|isamplerCubeArray|isampler1D|isampler1DArray|usampler[123]D|usamplerCube|usampler[12]DArray|usampler2DRect|usampler2DMS|usampler2DMSArray|usamplerCubeArray|usampler1D|usampler1DArray|image[123]D|imageCube|image[12]DArray|image2DRect|imageBuffer|image2DMS|image2DMSArray|imageCubeArray|image1D|image1DArray|iimage[123]D|iimageCube|uimage[123]D|uimageCube|atomic_uint)\\b",
          "name": "entity.name.class.shaderlab"
        },
        {
          "match": "\\b(radians|degrees|sin|cos|tan|asin|acos|atan|sinh|cosh|tanh|asinh|acosh|atanh|pow|exp|log|exp2|log2|sqrt|inversesqrt|abs|sign|floor|ceil|fract|mod|modf|min|max|clamp|mix|step|smoothstep|isnan|isinf|floatBitsToInt|floatBitsToUint|intBitsToFloat|uintBitsToFloat|fma|frexp|ldexp|packUnorm2x16|packUnorm4x8|packSnorm2x16|packSnorm4x8|unpackUnorm2x16|unpackUnorm4x8|unpackSnorm2x16|unpackSnorm4x8|packHalf2x16|unpackHalf2x16|packDouble2x32|unpackDouble2x32|length|distance|dot|cross|normalize|reflect|refract|faceforward|matrixCompMult|outerProduct|transpose|determinant|inverse|lessThan|lessThanEqual|greaterThan|greaterThanEqual|equal|notEqual|any|all|not|uaddCarry|usubBorrow|umulExtended|imulExtended|bitfieldExtract|bitfieldInsert|bitfieldReverse|bitCount|findLSB|findMSB|textureSize|textureQueryLod|textureQueryLevels|texture|textureProj|textureLod|textureOffset|texelFetch|texelFetchOffset|textureProjOffset|textureLodOffset|textureProjLod|textureProjLodOffset|textureGrad|textureGradOffset|textureProjGrad|textureProjGradOffset|textureGather|textureGatherOffset|textureGatherOffsets|textureSamples|atomicAdd|atomicMin|atomicMax|atomicAnd|atomicOr|atomicXor|atomicExchange|atomicCompSwap|atomicCounter|atomicCounterDecrement|atomicCounterIncrement|atomicCounterAdd|atomicCounterSubtract|atomicCounterMin|atomicCounterMax|atomicCounterAnd|atomicCounterOr|atomicCounterXor|atomicCounterExchange|atomicCounterCompSwap|imageSize|imageSamples|imageLoad|imageStore|imageAtomicAdd|imageAtomicMin|imageAtomicMax|imageAtomicAnd|imageAtomicOr|imageAtomicXor|imageAtomicExchange|imageAtomicCompSwap|dFdx|dFdy|dFdxFine|dFdyFine|dFdxCoarse|dFdyCoarse|fwidth|fwidthFine|fwidthCoarse|interpolateAtCentroid|interpolateAtSample|interpolateAtOffset|EmitStreamVertex|EndStreamPrimitive|EmitVertex|EndPrimitive|barrier|memoryBarrier|memoryBarrierAtomicCounter|memoryBarrierBuffer|memoryBarrierShared|memoryBarrierImage|groupMemoryBarrier)\\b",
          "name": "string.shaderlab"
        },
        {
          "match": "\\b(main)\\s*(?=\\()",
          "name": "entity.name.class.shaderlab"
        },
        {
          "match": "\\b(true|false)\\b",
          "name": "entity.name.class.shaderlab"
        }
      ]
    }
  }
}
```
Lastly, to change the color of your context, you can replace the `"name"` section of the built-in scope name to any of these. What the output will be is whatever theme you are using so it's possibly max 5 colors.
-Scope	                         Typical Color
-entity.name.class.shaderlab	   Teal/Green
-entity.name.function.shaderlab	 Teal/Green
-keyword.shaderlab.shaderlab     Light Blue
-support.function.shaderlab	     Light Blue
-storage.type.shaderlab	         Light Blue
-markup.heading.shaderlab        Brown/Orange
-string.shaderlab                Brown/Orange
-string.quoted.shaderlab         Brown/Orange
-constant.numeric.shaderlab      Light Green
-variable.parameter.shaderlab	   Grey
-markup.underline.shaderlab      Grey
-comment	                       Green

Now you can include all the GLSL shader files you need for syntax highlighting, have the context keywords light up, and change them to any color you want. Here is an example:
<img width="2040" height="1492" alt="shader" src="https://github.com/user-attachments/assets/dde80d36-9798-43ef-9e83-f0dc38a06b72" />

TL;DR, here is shaderlab.json file. 
1. Place it in: `C:\Program Files\Microsoft Visual Studio\18\community\common7\IDE\CommonExtensions\Microsoft\TextMate\Starterkit\Extensions\shaderlab\syntaxes\`
2. Remove your shader files from your project. (Don't delete them, just add them back in).
3. Restart Visual Studio 2026
4. Open your shader files



