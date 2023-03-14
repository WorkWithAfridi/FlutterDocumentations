# Publish Flutter Package : Documentation

---

# Start by:

> Package - contains only dart/flutter code, plugin contains native implementations
> 

1. Creating a new Flutter project via Android Studio/ VS Code 
> During the selection process change **Project type from application to package.**
*Follow “my_package” naming convention.*
2. Update **pubspec.yaml** file with the correct documentation.
3. Crate homepage via Github and link it to pubspec.yaml → homepage: //Your repo link here.
4. Update license:
//You can use the MIT license…
    
    <aside>
    💡 MIT LICENSE:
    
    PACKAGE NAME 
    
    Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions: 
    
    The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software. 
    
    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
    
    </aside>
    
5. Update README.md file with proper documentation.
6. Update CHANGELOG.md with the correct version number and changes-document.
7. Run “***dart pub publish —dry-run***” on terminal ← this will check for warnings and let us know
8. Then finally publish by running “***dart pub publish***”

**FOR Updating purposes:
Follow step 3-9 again.**