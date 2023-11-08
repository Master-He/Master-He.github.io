将python应用编译成小的exe, elf





`pyinstaller xxx.spec` 是运行 PyInstaller 并使用 xxx.spec` 文件进行打包的命令。

在使用 PyInstaller 打包程序时，你可以创建一个名为 `spec` 文件的配置文件，其中包含有关打包选项和参数的详细信息。`spec` 文件告诉 PyInstaller 如何处理你的程序，并指定要包含的文件、依赖项、数据文件等。

当你运行 `pyinstaller` 命令时，可以将 `spec` 文件作为参数传递给命令，例如 `pyinstaller xxx.spec`。这将告诉 PyInstaller 使用指定的 `spec` 文件进行打包操作，而不是使用默认的配置选项。

通过使用 `spec` 文件，你可以更精确地控制打包过程，自定义生成的可执行文件的行为和特性。可以在 `spec` 文件中设置诸如文件路径、导入模块、数据文件、图标、版本信息等选项。

创建 `spec` 文件的一种方法是使用 `pyi-makespec` 命令，它会根据你的程序自动生成一个初始的 `spec` 文件，然后你可以根据需要进行修改和自定义。

总而言之，`pyinstaller scanner.spec` 命令表示使用指定的 `scanner.spec` 文件来运行 PyInstaller 并进行打包操作，以根据该文件中的配置选项生成可执行文件。





pyinstaller spec文件示例

```python
# -*- mode: python ; coding: utf-8 -*-

block_cipher = None

a = Analysis(['your_script.py'],
             pathex=['/path/to/your/script'],
             binaries=[],
             datas=[],
             hiddenimports=[],
             hookspath=[],
             runtime_hooks=[],
             excludes=[],
             win_no_prefer_redirects=False,
             win_private_assemblies=False,
             cipher=block_cipher,
             noarchive=False)

pyz = PYZ(a.pure, a.zipped_data,
          cipher=block_cipher)

exe = EXE(pyz,
          a.scripts,
          a.binaries,
          a.zipfiles,
          a.datas,
          [],
          name='your_script',
          debug=False,
          bootloader_ignore_signals=False,
          strip=False,
          upx=True,
          upx_exclude=[],
          runtime_tmpdir=None,
          console=False,
          disable_windowed_traceback=False,
          target_arch=None,
          codesign_identity=None,
          entitlements_file=None )

coll = COLLECT(exe,
               a.binaries,
               a.zipfiles,
               a.datas,
               strip=False,
               upx=True,
               upx_exclude=[],
               name='dist')
```

