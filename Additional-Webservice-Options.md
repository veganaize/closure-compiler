# closure-compiler webservice options

In addition to the options provided in the [webservice UI](http://closure-compiler.appspot.com), the following options are also available:

**option**|**example**|**accepted values**
----------|-----------|-------------------
debug|// @debug true|true, false
warning level|// @warning_level VERBOSE|QUIET, DEFAULT, VERBOSE
externs url|// @externs_url http://a.url|
externs code|// @js_externs var externName;|
language|// @language ecmascript5|ECMASCRIPT3, ECMASCRIPT5, ECMASCRIPT5_STRICT, ECMASCRIPT6, ECMASCRIPT6_STRICT
language_out|// @language ecmascript5|ECMASCRIPT3, ECMASCRIPT5, ECMASCRIPT5_STRICT
use_types_for_optimization|// @use_types_for_optimization true|true, false
disable_property_renaming|// @disable_property_renaming true|true, false