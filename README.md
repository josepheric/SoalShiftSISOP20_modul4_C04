# SoalShiftSISOP20_modul4_C04

# Nomer 4
Pada Soal Nomer 4 diminta untuk membuat log pada FUSE yang dibuat.

Hal ini saya implementasikan dengan fungsi WriteLog
```
void writeLog (char *command,char* dir, int mode,char* desc)

{

  time_t t = time(NULL);

  struct tm tm = *localtime(&t);

  char log[1000];

  char* level;

  char*desc2="";

  int flag =0;

  if (strcmp (desc,"") != 0)

  {

    desc2 = desc;

    flag=1;

  }

  if (mode ==1)

  {

    level = "INFO";

  }

  else

  {

    level = "WARNING";

  }



  if (flag==1)

  {

    sprintf(log, "%s::%02d%02d%02d-%02d:%02d:%02d::%s::%s::%s", level, (tm.tm_year+1900)%100, (tm.tm_mon +1), (tm.tm_mday) %32, tm.tm_hour, tm.tm_min, tm.tm_sec, command, dir,desc2);

  }

  else

    sprintf(log, "%s::%02d%02d%02d-%02d:%02d:%02d::%s::%s", level, (tm.tm_year+1900)%100, (tm.tm_mon +1), (tm.tm_mday) %32, tm.tm_hour, tm.tm_min, tm.tm_sec, command, dir);





  FILE *out = fopen("/home/eric/fs.log", "a");

  fprintf(out, "%s\n", log);

  fclose(out);

}
```

Fungsi ini menerima 4 input parameter: command yang dijalankan, directory dijalankannya, mode, dan desc.

Mode digunakan untuk membedakan apakah command termasuk level INFO atau WARNING, berikut implementasinya:
```
 if (strcmp (desc,"") != 0)

  {

    desc2 = desc;

    flag=1;

  }

  if (mode ==1)

  {

    level = "INFO";

  }

  else

  {

    level = "WARNING";

  }
  ```
dan berikut adalah implementasi pembuatan fs.log:
```
 if (flag==1)

  {
//Bila punya deskripsi
    sprintf(log, "%s::%02d%02d%02d-%02d:%02d:%02d::%s::%s::%s", level, (tm.tm_year+1900)%100, (tm.tm_mon +1), (tm.tm_mday) %32, tm.tm_hour, tm.tm_min, tm.tm_sec, command, dir,desc2);

  }

  else
//bila tidak punya deskripsi
    sprintf(log, "%s::%02d%02d%02d-%02d:%02d:%02d::%s::%s", level, (tm.tm_year+1900)%100, (tm.tm_mon +1), (tm.tm_mday) %32, tm.tm_hour, tm.tm_min, tm.tm_sec, command, dir);





  FILE *out = fopen("/home/eric/fs.log", "a");

  fprintf(out, "%s\n", log);

  fclose(out);
  ```
  Fungsi-fungsi dibawah adalah fungsi yang dibutuhkan FUSE, dan dimodifikasi sedemikian rupa agar FUSE memeliki root ke /home/user/Documents. Tiap fungsi juga memanggil WriteLog untuk pembuatan lognya
  ```
  static int xmp_getattr(const char *path, struct stat *stbuf)

{

    writeLog("LS", path,1,nodesc);

    int res;



  char fpath[1000];



  sprintf(fpath,"%s%s",dirpath,path);



  res = lstat(fpath, stbuf);







  if (res == -1)



  return -errno;







  return 0;

}

static int xmp_readdir(const char *path, void *buf, fuse_fill_dir_t filler,

		       off_t offset, struct fuse_file_info *fi)

{

    writeLog("CD", path,1,nodesc);

    char fpath[1000];



  if(strcmp(path,"/") == 0)



  {



  path=dirpath;



  sprintf(fpath,"%s",path);



  }



  else sprintf(fpath, "%s%s",dirpath,path);



  int res = 0;







  DIR *dp;



  struct dirent *de;







  (void) offset;



  (void) fi;







  dp = opendir(fpath);



  if (dp == NULL)



  return -errno;







  while ((de = readdir(dp)) != NULL) {



  struct stat st;



  memset(&st, 0, sizeof(st));



  st.st_ino = de->d_ino;



  st.st_mode = de->d_type << 12;



  res = (filler(buf, de->d_name, &st, 0));



  if(res!=0) break;



  }







  closedir(dp);



  return 0;

}



static int xmp_read(const char *path, char *buf, size_t size, off_t offset,

		    struct fuse_file_info *fi)

{

    writeLog("CAT", path,1,nodesc);

    char fpath[1000];



  if(strcmp(path,"/") == 0)



  {



  path=dirpath;



  sprintf(fpath,"%s",path);



  }



  else sprintf(fpath, "%s%s",dirpath,path);



  int res = 0;



  int fd = 0 ;







  (void) fi;



  fd = open(fpath, O_RDONLY);



  if (fd == -1)



  return -errno;







  res = pread(fd, buf, size, offset);



  if (res == -1)



  res = -errno;







  close(fd);



  return res;

}



static int xmp_mkdir(const char *path, mode_t mode)

{



    int res;

        char fpath[1000];



        if (!strcmp(path, "/"))

        {

            path = dirpath;

            sprintf(fpath, "%s", path);

        }

        else

        {

            sprintf(fpath, "%s%s", dirpath, path);

        }

        res = mkdir(fpath, mode);

      writeLog("MKDIR", fpath,1,nodesc);

        if (res == -1)

            return -errno;



        return 0;

}



static int xmp_mknod(const char *path, mode_t mode, dev_t rdev)

{

    writeLog("CREATE", path,1,nodesc);

	int res;



	/* On Linux this could just be 'mknod(path, mode, rdev)' but this
	   is more portable */

	if (S_ISREG(mode)) {

		res = open(path, O_CREAT | O_EXCL | O_WRONLY, mode);

		if (res >= 0)

			res = close(res);

	} else if (S_ISFIFO(mode))

		res = mkfifo(path, mode);

	else

		res = mknod(path, mode, rdev);

	if (res == -1)

		return -errno;



	return 0;

}



static int xmp_unlink(const char *path)

{





    int res;

        char fpath[1000];



        if (!strcmp(path, "/"))

        {

            path = dirpath;

            sprintf(fpath, "%s", path);

        }

        else

        {

            sprintf(fpath, "%s%s", dirpath, path);

        }

        res = unlink(fpath);



          writeLog("REMOVE", fpath,2,nodesc);

        if (res == -1)

            return -errno;



        return 0;

}



static int xmp_rmdir(const char *path)

{



    int res;

       char fpath[1000];



       if (!strcmp(path, "/"))

       {

           path = dirpath;

           sprintf(fpath, "%s", path);

       }

       else

       {

           sprintf(fpath, "%s%s", dirpath, path);

       }



       res = rmdir(fpath);

        writeLog("RMDIR", fpath,2,nodesc);

       if (res == -1)

           return -errno;



       return 0;

}



static int xmp_rename(const char *from, const char *to)

{



    int res;

     char fpath[1000];

     char tpath[1000];

     if (!strcmp(from, "/"))

     {

         from = dirpath;

         sprintf(fpath, "%s", from);

     }

     else

     {

         sprintf(fpath, "%s%s", dirpath, from);

     }

     if (!strcmp(to, "/"))

     {

         to = dirpath;

         sprintf(tpath, "%s", to);

     }

     else

     {

         sprintf(tpath, "%s%s", dirpath, to);

     }

     res = rename(fpath, tpath);

     writeLog("RENAME", fpath,1,tpath);

     if (res == -1)

         return -errno;



     return 0;

}



static int xmp_truncate(const char *path, off_t size)

{



    int res;

       char fpath[1000];

       char name[1000];

       sprintf(name, "%s", path);

       sprintf(fpath, "%s%s", dirpath, name);

       res = truncate(fpath, size);

         writeLog("TRUNCATE", fpath,1,nodesc);

       if (res == -1)

           return -errno;



       return 0;



}





static int xmp_open(const char *path, struct fuse_file_info *fi)

{



    int res;

     char fpath[1000];



     if (!strcmp(path, "/"))

     {

         path = dirpath;

         sprintf(fpath, "%s", path);

     }

     else

     {

         sprintf(fpath, "%s%s", dirpath, path);

     }

     res = open(fpath, fi->flags);

      writeLog("OPEN", fpath,1,nodesc);

     if (res == -1)

         return -errno;



     close(res);

     return 0;

}



static int xmp_write(const char *path, const char *buf, size_t size,

		     off_t offset, struct fuse_file_info *fi)

{



    int fd;

      int res;

      char fpath[1000];



      if (!strcmp(path, "/"))

      {

          path = dirpath;

          sprintf(fpath, "%s", path);

      }

      else

      {

          sprintf(fpath, "%s%s", dirpath, path);

      }

      (void)fi;



      fd = open(fpath, O_WRONLY);

      if (fd == -1)

          return -errno;



        writeLog("WRITE", fpath,1,nodesc);

      res = pwrite(fd, buf, size, offset);

      if (res == -1)

          res = -errno;



      close(fd);

      return res;

}

static int xmp_chmod(const char *path, mode_t mode)

{



  int res;

     char fpath[1000];



     if (!strcmp(path, "/"))

     {

         path = dirpath;

         sprintf(fpath, "%s", path);

     }

     else

     {

         sprintf(fpath, "%s%s", dirpath, path);

     }



     res = chmod(fpath, mode);

       writeLog("CHMOD", fpath,1,nodesc);

     if (res == -1)

         return -errno;



     return 0;

}

static struct fuse_operations xmp_oper = {

	.getattr = xmp_getattr,

	.readdir = xmp_readdir,

	.read = xmp_read,

	.mkdir = xmp_mkdir,

	.mknod = xmp_mknod,

  .chmod		= xmp_chmod,

	.unlink = xmp_unlink,

	.rmdir = xmp_rmdir,

	.rename = xmp_rename,

	.truncate = xmp_truncate,

	.open = xmp_open,

	.read = xmp_read,

	.write = xmp_write,

};
```
Kesulitan dalam mengerjakan:
Nomer 1,2,3 terlalu kompleks
