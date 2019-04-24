**NTI has been removed from closure-compiler**

Here are some of the reasons for this:
1. There was no clear path to transitioning all users from OTI to NTI in the near future,
   leaving us stuck supporting 2 type checking code paths for the foreseeable future.
2. NTI wasn't a clear win.
   Although NTI caught some errors that OTI missed, it also missed some errors that OTI catches.
3. The NTI code didn't appear to be inherently easier to maintain and improve than the OTI code.
4. NTI was much more prone to insist that type annotations be added to existing code,
   making it harder for projects to start using the compiler.
5. Having 2 type checkers made all the things we most want to accomplish harder,
   because both type checkers would have to be modified for everything.

Deciding to drop NTI was not an easy decision, but it was a necessary one to unblock compiler development.

It did leave us with a backlog of "this works in NTI, so we won't fix it in OTI" issues, but we've already fixed some of the worst of these. (e.g. the strictCheckTypes flag)
We'll be fixing more in the near future as we work on OTI (now just "the type checker") to make it fully understands language features up through the current EcmaScript standard.
