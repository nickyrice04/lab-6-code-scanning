# Answers to Part 3

Add your answers to the questions in Part 3, Step 2 below. 

## Vulernability Remediation:
### Vulnerability 1: 
1. Which package or library are you addressing?

   PyYAML 5.1, pinned in requirements.txt. Both Trivy and Docker Scout flagged it as CRITICAL.

2. Which CVE is linked to this vulnerability?

   CVE-2020-14343 (CVSS 9.8). PyYAML versions before 5.4 can run arbitrary code when they load an untrusted YAML file with the full loader. PyGoat does exactly this in introduction/views.py line 553, where yaml.load(file, yaml.Loader) parses a file the user uploads. An attacker could upload YAML that calls a Python object like subprocess.Popen and run commands on the server.

3. What remediation steps do you suggest?

   Upgrade PyYAML to 5.4 or newer in requirements.txt and rebuild the image. The upgrade alone is not enough because yaml.Loader can still build arbitrary objects, so the code should also switch to yaml.safe_load(file), which only creates basic types like strings, lists and dictionaries. The same change applies to yaml.load(stream in introduction/lab_code/test.py.

### Vulnerability 2:
1. Which vulnerability are you addressing?

   Arbitrary code execution in Pillow 9.4.0 through PIL.ImageMath.eval. Both scanners flagged it as CRITICAL.

2. Which CVE is linked to this vulnerability?

   CVE-2023-50447 (CVSS 8.1 in Trivy, 9.3 in Docker Scout). In Pillow up to 10.1.0, ImageMath.eval evaluates its expression with Python's eval, and the environment parameter lets an attacker reach Python built-ins and run code. PyGoat calls ImageMath.eval(function_str, ...) in introduction/views.py line 581, and function_str comes straight from the request, so a user can send an expression that runs code on the server.

3. What remediation steps do you suggest?

   Upgrade Pillow to 10.2.0 or newer in requirements.txt and rebuild the image. Newer Pillow versions also add ImageMath.lambda_eval and ImageMath.unsafe_eval, which make the risk explicit. Beyond the upgrade, user input should never be passed to an eval style function. The view should offer a fixed list of allowed operations and map each one to code on the server instead of evaluating the string the user sends.
