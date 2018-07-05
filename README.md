## Day18_180705

- 좋아요 버튼 + 개수 넣고 변화하는것 까지
- 별점주기 (좋아요버튼에 컬럼을 하나 추가하면 된다.  t.integer :score   )
- 댓글 (Ajax) 입력 + 삭제 + 수정

  - 댓글입력시 글자제한( front + backend )
- pagination (kaminari)
- hastag (제목중복, 아이디중복 찾을 때 ) 자동완성기능?



### Ajax 복습

- MVVM : 요새 핫한 프론트엔드 자바스크립트 프레임워크
- 모든화면을 Ajax라고 만들기엔 서버를 항상부르기엔 불편해서 만들어진 기술
- 처음 `show.html.erb` 에서 url, method, data 를 지정해주고, 이 요청을 받아드릴 `routes` 에 해당 url을 컨트롤러에 지정
-  `movies_controller` 에서 좋아요랑 좋아요 취소 로직을 설정해주었다. 
- 액션명과 같은 js를 써주어 서버가 movies_controller에서 like_movie 액션이 동작할때 해당 자바스크립트가 실행되도록한다.



### 좋아요 버튼 + 개수

*show.html.erb*

```ruby
<h1><%= @movie.title %></h1>
<hr>
<p> <%=@movie.description %></p>
<hr>

<%= link_to 'Edit', edit_movie_path(@movie) %> |
<%= link_to 'Back', movies_path %>  | 
<% if @user_likes_movie.nil? %>
<button class="btn btn-info like">좋아요 (<span class="like-count"><%= @movie.likes.count %></span>)</button>
<% else %>
<button class="btn btn-warning like">좋아요 취소 (<span class="like-count"><%= @movie.likes.count %></span>)</button>
<% end %>
<script>
    $(document).on('ready',function(){
        $('.like').on('click', function(){
            console.log("like~!") // 확인용
            $.ajax({
               url: '/likes/<%= @movie.id %>' 
            });
        })
    });
</script>
```

*like-movie.js.erb*

```javascript
var like_count = parseInt($('.like-count').text());
console.log(like_count);
//좋아요가 취소된 경우
// 좋아요 취소버튼 -> 좋아요 버튼으로 바꿔준다.
if (<%= @like.frozen? %>) {
    like_count  = like_count - 1
    $('.like').html(`좋아요(<span class='like-count'>${like_count}</span>)`).removeClass('btn-warning').addClass('btn-info'); // string Interpolation
}else{
//좋아요가 새로눌린 경우
// 좋아요 버튼 -> 좋아요 취소버튼으루~!~!
    alert("좋아요 설정되쩡!");
    like_count = like_count + 1
    $('.like').html(`좋아요 취소(<span class='like-count'>${like_count}</span>)`).removeClass('btn-info').addClass('btn-warning');
}
```

> => parseInt($('.like-count').text());
>
> => string Interpolation 얘는 `로 쓰면 여러줄도 가능 (줄바꿈가능)
>
> 변수명에 - 은 안된다. _ (under score은 가능)

*movies_controller.rb*

```ruby
 def show
    if user_signed_in?
      @user_likes_movie = Like.where(user_id: current_user.id, movie_id: @movie.id).first
    else
      @user_likes_movie =[]
 end
```



### 댓글 입력 & 삭제

- 댓글을 입력받을 폼을 작성

*~/watcha_app/app/views/movies/show.html.erb*

```ruby
...
<form class="text-right comment">
    <input class="form-control comment-contents"><<br/>
    <input type="submit" value="댓글쓰기" class="btn btn-success">
</form

<ul class="list-group comment-list">
    <!--<li class="list-group-item">Cras justo odio</li>-->
</ul>
...
```



- form(요소)이 제출(이벤트)될때 (이벤트 리스너)

*~/watcha_app/app/views/movies/show.html.erb*

```ruby
...
<script>
    $(document).on('ready',function(){
        $('.like').on('click', function(){
            console.log("like~!") // 확인용
            $.ajax({
               url: '/likes/<%= @movie.id %>' 
            });
        })
        $('.comment').on('submit', function(e){
            e.preventDefault();   
            # 실제로<input type="submit"> 여기에서 page redirection이 진행되어 기존 댓글이 사라진다. 그러므로 이를 페이지 리디렉션을 방지하기위해 사용. 값이 log로 넘어오는것을 볼수있다.
            var comm = $('.comment-contents').val();
            console.log(comm);
        });
    });
</script>
...
```



- form에 input(요소) 안에 있는 내용물(메소드)을 받아서

[jquery add element to element][http://api.jquery.com/append/]

*~/watcha_app/app/views/movies/show.html.erb*

```ruby
<script>
    $(document).on('ready',function(){
        $('.like').on('click', function(){
            console.log("like~!") // 확인용
            $.ajax({
               url: '/likes/<%= @movie.id %>' 
            });
        })
        $('.comment').on('submit', function(e){
            e.preventDefault();
            var comm = $('.comment-contents').val();
            console.log(comm);
            $('.comment-list').append(comm);
        });
    });
</script>
```

한줄씩 쓰게 하기 위해서 다음의 코드로 수정한다.

```겨ㅠㅛ
 $('.comment-list').append(`<li class="list-group-item">${comm}</li>`);
 $('.comment-contents').val(''); // 댓글입력창에 존재하는 값 날리기용
```

하지만, 첫번째 쓴것을 맨위에 올리기위해 `.append` 가 아닌 `.prepend`로 사용한다.

- *comment model 만들기*
  - column : user_id, movie_id, contents
  - association :
    - movie(1) - comment (N)
    - user(1) - comment(N)
  - url("/movies/:movie_id/comments", method: post)인 ajax 코드 짜보기

```ruby
<h1><%= @movie.title %></h1>
<hr>
<p> <%=@movie.description %></p>
<hr>

<%= link_to 'Edit', edit_movie_path(@movie) %> |
<%= link_to 'Back', movies_path %>  | 
<% if @user_likes_movie.nil? %>
<button class="btn btn-info like">좋아요 (<span class="like-count"><%= @movie.likes.count %></span>)</button>
<% else %>
<button class="btn btn-warning like">좋아요 취소 (<span class="like-count"><%= @movie.likes.count %></span>)</button>
<% end %>
<hr>

<form class="text-right comment">
    <input class="form-control comment-contents"><br/>
    <input type="submit" value="댓글쓰기" class="btn btn-success">
</form>
<hr>
<h3>댓글</h3>
<ul class="list-group comment-list">
    <!--<li class="list-group-item">Cras justo odio</li>-->
</ul>
<hr>
<script>
    $(document).on('ready',function(){
        $('.like').on('click', function(){
            console.log("like~!") // 확인용
            $.ajax({
               url: '/likes/<%= @movie.id %>' 
            });
        })
        $('.comment').on('submit', function(e){
            e.preventDefault();
            var comm = $('.comment-contents').val();
            console.log(comm);
            $.ajax({
                url: '/movies/<%= @movie.id %>/comments',
                data: {contents: comm},
                method: 'post'
            });
            $('.comment-list').prepend(`<li class="list-group-item">${comm}</li>`);
            $('.comment-contents').val(''); // 값날리기용
        });
        
        
    });
</script>


```

 ! 라우팅 참고 =>  [Nested Resources](http://guides.rubyonrails.org/routing.html#nested-resources)

*routes.rb*

```ruby
    member do
      get '/comments' => 'movies#create_comment'
    end
```

=> comments_movie GET    /movies/:id/comments(.:format) movies#create_comment 

이렇게 된다.

`AbstractController::ActionNotFound (The action 'create_comment' could not be found for MoviesController): ` controller에 액션이 없기 때문에 에러일것이다.

- 보낼때에는 내용물, 현재보고있는 movie의 id값도 같이 보낸다

*movies_controller*

```ruby
 before_action :set_movie, only: [:show, :edit, :update, :destroy, :create_comment]

  def create_comment
    #  @movie = Movie.find(params[:id]) <- set_movie 랑 같으니까
    @comment =Comment.create(user_id: current_user.id, movie_id: @movie.id, contents: params[:contents])
      	   #  @comment =Comment.create(user_id: current_user.id, movie_id: @movie.id)랑
           # movie.comments.new(user_id: current_user.id).save 와 같은 말
  end
```

이렇게 컨트롤러에서 넘기는 contents 는 *show.html.erb* 에 있는 `  data: {contents: comm},` 라는 Ajax의 코드로 데이터가 넘어온다.



- 서버에서 저장하고 response로 보내줄 js.erb 파일을 작성한다.

*~/watcha_app/app/views/movies/create_comment.js.erb* 파일을 생성하고, 안에 *show.html.erb*에 있던 댓글 작성기능 코드를 잘라온다

```ruby
$('.comment-list').prepend(`<li class="list-group-item"><%= @comment.contents %></li>`);
$('.comment-contents').val(''); // 값날리기용
alert('댓글 등록이 완료되었습니다.');
```



- js.erb 파일에서는 댓글이 표시될 영역에 등록된 댓글의 내용을 추가해준다.

*show.html.erb* 에 기존에 등록되어있는 댓글을 출력하는 코드

```ruby
<ul class="list-group comment-list">
    <!--기존에 등록되어있는 댓글 출력-->
    <% @movie.comments.reverse.each do |comment| %>
        <li class="list-group-item d-flex justify-content-between"><%= comment.contents %> ( <%= comment.user.email %> )
    <button class="btn btn-danger">삭제</button></li>
    <% end %>
</ul>
```

[정렬-부트스트랩][https://getbootstrap.com/docs/4.1/utilities/flex/#justify-content]

- 댓글에 있는 삭제버튼(요소)을 누르면(이벤트 리스너) 해당 댓글이 눈에 안보이게 되고, (이벤트 핸들러)

  *views/movies/show.html.erb*

```ruby
<script>
    ...
		$('.destroy-comment').on('click', function(){
            console.log("destroyed!!!!");   
            $(this).parent().remove();
        });
	...
</script>	
```

> button을 감싸는 태그의 바로 상위태그(<li>) 불러오는거 => parent
>
> 상위태그 모두 다 불러오려면 =>parents 
>
> parents().find() 이런식으로도 쓸수는 있겠지.

- 실제 db에서도 삭제가 된다.(ajax코드)

  ```ruby
  <ul class="list-group comment-list">
      <!--<li class="list-group-item">Cras justo odio</li>-->
      <!--기존에 등록되어있는 댓글 출력-->
      <% @movie.comments.reverse.each do |comment| %>
          <li class="list-group-item d-flex justify-content-between">
              <%= comment.contents %> ( <%= comment.user.email %> )
          <button data-id="<%= comment.id %>" class="btn btn-danger destroy-comment">삭제</button></li>
      <% end %>
  </ul>
  <hr>
  <script>
  ...
          $('.destroy-comment').on('click', function(){
              console.log("destroyed!!!!");   
              $(this).parent().remove();
              var comment_id = $(this).attr('data-id');
            //  $(this).data('id'); //이렇게도 뽑아낼수있으data-id로 썼던 이유.
            console.log(comment_id);
              $.ajax({
                  url: "/movies/comments/" + comment_id //resouceful routing,
                  method: "delete"
              })
          });
      });
  </script>
  ```

  > GET https://my-second-rails-app-binn02.c9users.io/movies/comments/4 404 (Not Found) 라는 에러뜬다.
  >
  > 왜냐하면 저 url로 연결되는게 없기 때문이다.

  *routes.rb*

  ```ruby
      collection do
        delete '/comments/:comment_id' => 'movies#destroy_comment'
      end
  ```

  > 이러면 또 액션이 없어서 에러난다.

  *movies_controller.rb*

  ```ruby
    def destroy_comment
       @comment = Comment.find(params[:comment_id]).destroy
    end
  ```

  액션이 실행될때만 js를 돌려주기 위해서 파일을 분리하자

  *destroy_comment.js.erb*

  ```ruby
  alert("댓글이 삭제되었습니다.");
  $('.comment-<%=@comment.id %>').remove();
  ```

  *show.html.erb*

  ```ruby
  <li class="comment-<%= comment.id %> // parent로 안하고 바로 받기위해서 class설정
  ```

- 새로 등록하는 댓글들에는 삭제버튼이 안나오는 에러를 해결하자

*~/watcha_app/app/views/movies/create_comment.js.erb*

```ruby
$('.comment-list').prepend(`<li class="comment-<%= @comment.id %> list-group-item d-flex justify-content-between">
            <%= @comment.contents %> ( <%= @comment.user.email %> )
            <button data-id="<%= @comment.id %>" class="btn btn-danger destroy-comment">삭제</button>
            </li>`);
$('.comment-contents').val(''); // 값날리기용
alert('댓글 등록이 완료되었습니다.');
```

> 아 근데 또 문제있어
>
> 새로 등록한 댓글에 삭제버튼이 동작하지 않는다. <- 자바스크립트로 추가된 친구는  `$('.destroy-comment').on('click', function(){` 얘가 동작하지 않는다

*show.html.erb*

```ruby
  $(document).on('click','.destroy-comment', function(){ 
            console.log("destroyed!!!!");   
            var comment_id = $(this).attr('data-id');
          //  $(this).data('id'); //이렇게도 뽑아낼수있으data-id로 썼던 이유.
          console.log(comment_id);
            $.ajax({
                url: "/movies/comments/" + comment_id, //resouceful routing
                method: "delete"
            })
        });
```

> `  $(document).on('click','.destroy_comment', function(){ ` : dom-tree를 다시 읽어서 '.destroy_comment'를 찾자. (쉼표로 끊어주었기에 목적지를 지정한거라 생각하자 )
>
> 이거는 *create_comment.js.erb* 에서 버튼의 클래스가 ` class="btn btn-danger destroy-comment` 이기때문에!



### 댓글수정

```ruby
<li class="comment-<%= comment.id %> list-group-item d-flex justify-content-between">
   <span class="comment-detail"><%= comment.contents %></span>
     <div>
      <button data-id="<%= comment.id %>" class="btn btn-warning text-white edit-comment">수정</button>
       <button data-id="<%= comment.id %>" class="btn btn-danger destroy-comment">삭제</button>
     </div>
</li>
                        
...
                        
 <script>
	..
	$('.edit-comment').on('click', function() {
           var detail = $('.comment-detail').text();
           console.log(detail);
        });
    ..
</script>
```

> 이거는 모든 댓글들을 다 합쳐서 보여준다



- 수정버튼(요소)을 클릭하면 (이벤트 리스너) 댓글이 있던 부분이 입력창으로 바뀌면서 원래 있던 댓글의 내용이 입력창에 들어간다.(이벤트 핸들러)

- 수정버튼은 확인 버튼으로 바뀐다.

*show.html.erb*

```ruby
<li class="comment-<%= comment.id %> list-group-item d-flex justify-content-between">
  <span class="comment-detail-<%= comment.id%>"><%= comment.contents %></span>
      <div>
        <button data-id="<%= comment.id %>" class="btn btn-warning text-white edit-comment">수정</button>
        <button data-id="<%= comment.id %>" class="btn btn-danger destroy-comment">삭제</button>
      </div>
</li> 
...
<script>
  $(document).on('click','.edit-comment', function() { //수정버튼(요소)을 클릭하면
       
        var comment_id = $(this).data('id');
        var edit_comment = $(`.comment-detail-${comment_id}`);
        var contents = edit_comment.text().trim();
        edit_comment.html(`
        <input type="text" value="${contents}" class="form-control edit-comment-${comment_id}">`);
            //(이벤트 리스너) 댓글이 있던 부분이 입력창으로 바뀌면서원래 있던 댓글의 내용이 입력창에 들어간다.(이벤트 핸들러)
          $(this).text("확인").removeClass("edit-comment btn-warning").addClass("update-comment btn-dark"); //수정버튼은 확인 버튼으로 바뀐다.
        });
...               
</script>
```

>     //   console.log($(this).parent().parent().find('.comment-detail'))); //li 에서 comment-detail을 찾기위해서
>     //  하지만 넘 거지같으므로  <span class="comment-detail-<%= comment.id%>"> 를 걍 해준다



---------



- 내용 수정 후 확인버튼을 클릭하면 입력창에 있던 내용물이 댓글의 원래 형태로 바뀌고 확인버튼은 다시 수정버튼으로 바뀐다.

```ruby
<script>
    ...
	  $(document).on('click','.update-comment' ,function(){ // 내용 수정 후 확인버튼을 클릭하면
           console.log("update"); 
          var comment_id = $(this).data('id');
          var comment_form = $(`.edit-comment-${comment_id}`)
        //  console.log(comment_form.val());
          var edit_comment = $(`.comment-detail-${comment_id}`);
          edit_comment.html(comment_form.val()); //입력창에 있던 내용물이 댓글의 원래 형태로 바뀌고
            $(this).text("수정").removeClass("update-comment btn-dark").addClass("edit-comment btn-warning"); //확인버튼은 다시 수정버튼으로 바뀐다.
        });
	...
</script>
```



- 입력창에 있던 내용물을 Ajax로 서버에 요청을 보낸다.

```ruby
       $.ajax({
                url: "/movies/comments/" + comment_id,
                data:{contents: comment_form.val()},
                method: "PATCH"
            })
```

> 404 (Not Found) 에러 발생 한다.

- 서버에서는 해당 댓글을 찾아 내용을 업데이트 한다.

*routes.rb*

```ruby
    collection do
      patch '/comments/:comment_id' => 'movies#update_comment'
    end
```

*movie_controller.rb*

```ruby
  def update_comment
    @comment = Comment.find(params[:comment_id])
    @comment.update(contents: params[:contents])
  end
```

> `@comment = Comment.find(params[:comment_id]).update(contents: params[:contents])` 로 하면 
>
> @comment에 T / F 형태로 담긴다.

*update_comment.js.erb*

```ruby
alert("수정이 완료되었습니다.");
var edit_comment = $('.comment-detail-<%=@comment.id%>');
edit_comment.html('<%= @comment.contents %>');
$('.update-comment').text("수정").removeClass("update-comment btn-dark").addClass("edit-comment btn-warning");
```

-------------

#### 수정 문제 2가지

1. 새글에 수정버튼이 없다

*show.html.erb*

```ruby
 <li class="comment-<%= comment.id %> list-group-item d-flex justify-content-between">
           <span class="comment-detail-<%= comment.id%>"><%= comment.contents %></span>
            <div>
                <button data-id="<%= comment.id %>" class="btn btn-warning text-white edit-comment">수정</button>
                <button data-id="<%= comment.id %>" class="btn btn-danger destroy-comment">삭제</button>
            </div>
</li>    
```

이 부분을 복사해서 *create_comment.js.erb* 에서 지역변수로만 바꿔서 적용해주자

*create_comment.js.erb*

```ruby
$('.comment-list').prepend(`<li class="comment-<%= @comment.id %> list-group-item d-flex justify-content-between">
           <span class="comment-detail-<%= @comment.id%>"><%= @comment.contents %></span>
            <div>
                <button data-id="<%= @comment.id %>" class="btn btn-warning text-white edit-comment">수정</button>
                <button data-id="<%= @comment.id %>" class="btn btn-danger destroy-comment">삭제</button>
            </div>
        </li>`);
$('.comment-contents').val(''); // 값날리기용
alert('댓글 등록이 완료되었습니다.');
```

> 사실 이것을 렌더로 처리하면 더 쉽고 간편해진다!!
>
> interpolation을 안써도 멀티라인으로 코드를 쓰려면 `` 를 써야한다.



2. 수정버튼이 여러개 열린다.

- 수정 버튼을 누르면 
- 전체 문서 중에서 `update-comment` 클래스를 가진 버튼 이 있는 경우 진행을 이벤트 핸들러를 끝내고 `return false;`, 하지 말자 라고 alert을 띄어주자

```ruby
$(document).on('click','.edit-comment', function() {
	var comment_id = $(this).data('id');
	var edit_comment = $(`.comment-detail-${comment_id}`);
	var contents = edit_comment.text().trim();
	var confirm_button = $('.update-comment').length;
	console.log(confirm_button);
        if(confirm_button > 0){
            alert("수정 버튼은 한개만 눌러주세요.");
            return false;
        }else{
        edit_comment.html(`
        <input type="text" value="${contents}" class="form-control edit-comment-${comment_id}">`);
        $(this).text("확인").removeClass("edit-comment btn-warning").addClass("update-comment btn-dark");    
        }});
```







#### ORM을 사용하는 이유 

: db에 있는 친구들을 객체로 쓰듯이 쓰려고 ! 

- 현재유저가 등록한 댓글들

  : current_user.comments

- 영화 하나가 가지고 있는 댓글들

  : movie.comments

- 코멘트 하나에 달린 무비

  : comment.movie
