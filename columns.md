`SettableColumn`:
    - https://github.com/saulpw/visidata/issues/413
    - SettableColumn's putValue adds rows and columns to sheet.cache, instead of sheet.rows. copyRows grabs the cliprows from sheet.rows. as a result of this, if you try to copy a SettableColumn's cells, you will get blank values.
