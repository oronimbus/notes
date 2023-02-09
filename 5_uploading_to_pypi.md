# Uploading to PyPi

1) Uploading to PyPi is very easy, first create your account and verify the email address.
2) Make sure your package is neatly structured and contains: `setup.cfg`,`setup.py`, `LICENSE.txt` and `README.md` (optional)
3) Run `python setup.py sdist bdist_wheel`, this will package the project
4) Upload using `twine`, if not installed just `pip install twine` and then `twine upload dist/*`
5) Now you can `pip install PACKAGE`!