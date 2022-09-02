# flask-8-note-post-update
home页变成了post页；添加一个new post的页面，并且可以修改内容

1. 添界面 - routes.py

      @app.route("/post/<int:post_id>") 建立post 页面
      def post(post_id):
          post = Post.query.get_or_404(post_id)
          return render_template('post.html', title=post.title, post=post)

      @app.route("/post/<int:post_id>/update", methods=["GET", "POST"])
      @login_required     必须登录，是自己的内容才能修改
      def update_post(post_id):
          post = Post.query.get_or_404(post_id)
          if post.author != current_user:
              abort(403)
          form = PostForm()   更新的话，换内容
          if form.validate_on_submit():
              post.title=form.title.data
              post.content=form.content.data
              db.session.commit()
              flash('Your post has been updated!', 'success')
              return redirect(url_for('post', post_id=post.id))
          elif request.method == 'GET':   先走这里，把信息存好
              form.title.data = post.title
              form.content.data = post.content
          return render_template('create_post.html', title='Update Post', 
              form=form, legend='Update Post')


      @app.route("/post/<int:post_id>/delete", methods=["POST"]) 删除选项
      @login_required
      def delete_post(post_id): 与上文一致
          post = Post.query.get_or_404(post_id)
          if post.author != current_user:
              abort(403)
          db.session.delete(post)
          db.session.commit() 数据修改
          flash('Your post has been deleted!', 'success') 页面提示
          return redirect(url_for('home'))


2. 文本编辑界面 - post.html
基本和create post一样，但是这边其实是一个新表，先copy再修改不同
显示图像+日期
分为内容部分和model部分

         <article class="media content-section">
          <img src="{{ url_for('static', filename='profile_pics/' + post.author.image_file) }}" class="rounded-circle article-img"> 显示头像
            <div class="media-body">
              <!-- <div class="article-metadata"> -->
                <a class="mr-2" href="#">{{ post.author.username }}</a> 引用用户名字
                <small class="text-muted">{{ post.date_posted.strftime('%Y-%m-%d') }}</small> 显示年月日
                {% if post.author == current_user %}
                    <div>
                        <a class="btn btn-secondary btn-sm mt-1 mb-1" href="{{url_for('update_post', post_id=post.id)}}">Update Post</a> 
                        <button type="button" class="btn btn-danger btn-sm m-1" data-bs-toggle="modal" data-bs-target="#deleteModal">Delete</button>    引用了bootstrap里面的模版来套用modal
                    </div>
                {% endif %}
                <!-- </div> -->       
              <h2 class="artitle-title" >{{ post.title }}</h2>
              <p class="article-content">{{ post.content }}</p>
            </div>
        </article> 
        <!-- Modal -->          下面是model-弹窗的设置
        <div class="modal fade" id="deleteModal" tabindex="-1" aria-labelledby="deleteModalLabel" aria-hidden="true">
            <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                <h5 class="modal-title" id="deleteModalLabel">Delete Post?</h5>  清除吗？
                
                </div>
                <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>  关闭
                <form action="{{ url_for('delete_post', post_id=post.id)}}" method="POST">
                    <input class="btn btn-danger" type="submit" value="Delete"> 
                </form>
                </div>
            </div>
            </div>
        </div>
