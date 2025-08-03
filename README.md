import { useState, useRef, useEffect } from 'react';
import { Heart, MessageCircle, Send, Moon, Sun, User, Settings, MoreHorizontal, Trash, Ban, Shield, X, Home, Bell, Image, XCircle } from 'lucide-react';
import { Button } from "/components/ui/button";
import { Card, CardContent, CardFooter, CardHeader } from "/components/ui/card";
import { Textarea } from "@/components/ui/textarea";
import { Input } from "@/components/ui/input";
import { Avatar, AvatarFallback, AvatarImage } from "/components/ui/avatar";
import { motion, AnimatePresence } from 'framer-motion';

type Comment = {
  id: string;
  text: string;
  timestamp: Date;
  userId: string;
};

type Post = {
  id: string;
  content: string;
  timestamp: Date;
  likes: number;
  liked: boolean;
  comments: Comment[];
  showComments: boolean;
  isUserPost: boolean;
  userId: string;
  imageUrl?: string;
};

type ServerMessage = {
  id: string;
  text: string;
  timestamp: Date;
};

type View = 'feed' | 'settings';

export default function LhymssSecret() {
  const [posts, setPosts] = useState<Post[]>([
    {
      id: '1',
      content: 'Just finished building my new project with React and TypeScript! So excited to share it with everyone.',
      timestamp: new Date(Date.now() - 3600000),
      likes: 24,
      liked: false,
      comments: [
        { id: 'c1', text: 'Looks amazing! Great job!', timestamp: new Date(Date.now() - 1800000), userId: 'user2' },
        { id: 'c2', text: 'Can you share the repo link?', timestamp: new Date(Date.now() - 1200000), userId: 'user3' }
      ],
      showComments: false,
      isUserPost: false,
      userId: 'user1'
    },
    {
      id: '2',
      content: 'Working on a new design system for our company. Tailwind CSS makes it so much easier!',
      timestamp: new Date(Date.now() - 7200000),
      likes: 42,
      liked: true,
      comments: [
        { id: 'c3', text: 'The color palette is perfect!', timestamp: new Date(Date.now() - 3600000), userId: 'user4' }
      ],
      showComments: false,
      isUserPost: true,
      userId: 'currentUser',
      imageUrl: 'https://images.unsplash.com/photo-1555066931-4365d14bab8c?w=800&auto=format&fit=crop'
    }
  ]);
  
  const [newPostContent, setNewPostContent] = useState('');
  const [newComment, setNewComment] = useState<Record<string, string>>({});
  const [darkMode, setDarkMode] = useState(false);
  const [currentView, setCurrentView] = useState<View>('feed');
  const [username, setUsername] = useState('Anonymous');
  const [newUsername, setNewUsername] = useState('Anonymous');
  const [menuOpen, setMenuOpen] = useState<string | null>(null);
  const [commentMenuOpen, setCommentMenuOpen] = useState<string | null>(null);
  const [avatar, setAvatar] = useState<string | null>(null);
  const [passcode, setPasscode] = useState('');
  const [developerMode, setDeveloperMode] = useState(false);
  const [showPasscodeInput, setShowPasscodeInput] = useState(false);
  const [userId] = useState(`#${Math.floor(1000 + Math.random() * 9000)}`);
  const [serverMessages, setServerMessages] = useState<ServerMessage[]>([]);
  const [newServerMessage, setNewServerMessage] = useState('');
  const [showServerMessageInput, setShowServerMessageInput] = useState(false);
  const [newPostImage, setNewPostImage] = useState<string | null>(null);
  const [newPostImagePreview, setNewPostImagePreview] = useState<string | null>(null);
  const fileInputRef = useRef<HTMLInputElement>(null);
  const imageInputRef = useRef<HTMLInputElement>(null);

  // Handle server message expiration
  useEffect(() => {
    const timers = serverMessages.map(message => 
      setTimeout(() => {
        setServerMessages(prev => prev.filter(m => m.id !== message.id));
      }, 15000)
    );
    
    return () => timers.forEach(timer => clearTimeout(timer));
  }, [serverMessages]);

  const handlePostSubmit = () => {
    if (newPostContent.trim() === '' && !newPostImage) return;
    
    const newPost: Post = {
      id: Date.now().toString(),
      content: newPostContent,
      timestamp: new Date(),
      likes: 0,
      liked: false,
      comments: [],
      showComments: false,
      isUserPost: true,
      userId: 'currentUser',
      imageUrl: newPostImage || undefined
    };
    
    setPosts([newPost, ...posts]);
    setNewPostContent('');
    setNewPostImage(null);
    setNewPostImagePreview(null);
  };

  const handleLike = (postId: string) => {
    setPosts(posts.map(post => {
      if (post.id === postId) {
        return {
          ...post,
          likes: post.liked ? post.likes - 1 : post.likes + 1,
          liked: !post.liked
        };
      }
      return post;
    }));
  };

  const toggleComments = (postId: string) => {
    setPosts(posts.map(post => 
      post.id === postId 
        ? { ...post, showComments: !post.showComments } 
        : post
    ));
  };

  const handleCommentChange = (postId: string, text: string) => {
    setNewComment({ ...newComment, [postId]: text });
  };

  const handleAddComment = (postId: string) => {
    const commentText = newComment[postId]?.trim();
    if (!commentText) return;
    
    const newCommentObj: Comment = {
      id: `c${Date.now()}`,
      text: commentText,
      timestamp: new Date(),
      userId: 'currentUser'
    };
    
    setPosts(posts.map(post => {
      if (post.id === postId) {
        return {
          ...post,
          comments: [...post.comments, newCommentObj]
        };
      }
      return post;
    }));
    
    setNewComment({ ...newComment, [postId]: '' });
  };

  const handleSaveUsername = () => {
    setUsername(newUsername);
    setCurrentView('feed');
  };

  const handleDeletePost = (postId: string) => {
    setPosts(posts.filter(post => post.id !== postId));
    setMenuOpen(null);
  };

  const handleDeleteAllPosts = () => {
    setPosts(posts.filter(post => post.isUserPost));
    setMenuOpen(null);
  };

  const handleBanUser = (userId: string) => {
    alert(`User ${userId} has been banned`);
    setMenuOpen(null);
  };

  const handleDeleteComment = (postId: string, commentId: string) => {
    setPosts(posts.map(post => {
      if (post.id === postId) {
        return {
          ...post,
          comments: post.comments.filter(comment => comment.id !== commentId)
        };
      }
      return post;
    }));
    setCommentMenuOpen(null);
  };

  const handleAvatarChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (event) => {
        if (event.target?.result) {
          setAvatar(event.target.result as string);
        }
      };
      reader.readAsDataURL(file);
    }
  };

  const triggerFileInput = () => {
    fileInputRef.current?.click();
  };

  const handlePasscodeSubmit = () => {
    if (passcode === '1234') {
      setDeveloperMode(true);
      setShowPasscodeInput(false);
      setCurrentView('feed');
      setPasscode('');
    } else {
      alert('Incorrect passcode');
    }
  };

  const goToHome = () => {
    setCurrentView('feed');
  };

  const handleServerMessageSubmit = () => {
    if (newServerMessage.trim() === '') return;
    
    const message: ServerMessage = {
      id: `msg${Date.now()}`,
      text: newServerMessage,
      timestamp: new Date()
    };
    
    setServerMessages([message, ...serverMessages]);
    setNewServerMessage('');
    setShowServerMessageInput(false);
  };

  const handleImageChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (event) => {
        if (event.target?.result) {
          setNewPostImagePreview(event.target.result as string);
        }
      };
      reader.readAsDataURL(file);
    }
  };

  const triggerImageInput = () => {
    imageInputRef.current?.click();
  };

  const removeImage = () => {
    setNewPostImagePreview(null);
    setNewPostImage(null);
    if (imageInputRef.current) {
      imageInputRef.current.value = '';
    }
  };

  const handleImageUpload = () => {
    if (newPostImagePreview) {
      setNewPostImage(newPostImagePreview);
    }
  };

  const formatTime = (date: Date) => {
    const now = new Date();
    const diffMs = now.getTime() - date.getTime();
    const diffMins = Math.floor(diffMs / 60000);
    const diffHours = Math.floor(diffMs / 3600000);
    const diffDays = Math.floor(diffMs / 86400000);
    
    if (diffMins < 1) return 'Just now';
    if (diffMins < 60) return `${diffMins}m ago`;
    if (diffHours < 24) return `${diffHours}h ago`;
    return `${diffDays}d ago`;
  };

  // Button styling based on mode
  const buttonClass = darkMode 
    ? "bg-blue-600 hover:bg-blue-700 text-white" 
    : "bg-black hover:bg-gray-800 text-white";

  const ghostButtonClass = darkMode 
    ? "hover:bg-blue-600/20 text-white" 
    : "hover:bg-black/10 text-gray-900";

  const outlineButtonClass = darkMode 
    ? "border border-blue-500 text-white hover:bg-blue-600" 
    : "border border-black text-black hover:bg-black hover:text-white";

  // Settings-specific button classes
  const settingsButtonClass = darkMode 
    ? "bg-blue-600 hover:bg-blue-700 text-white" 
    : "bg-black hover:bg-gray-800 text-white";

  const settingsOutlineButtonClass = darkMode 
    ? "border border-blue-500 bg-blue-600 text-white hover:bg-blue-700" 
    : "border border-black text-black hover:bg-black hover:text-white";

  // Get username with ID or "You" for current user
  const getUsernameDisplay = (userId: string, isCurrentUser: boolean = false, isPost: boolean = true) => {
    if (isCurrentUser) {
      return isPost ? (
        <span>
          {username} <span className="text-gray-500 dark:text-gray-400">(You)</span>
        </span>
      ) : (
        username
      );
    }
    
    // For other users, generate a consistent ID
    const idNumber = `#${Math.abs(userId.split('').reduce((a, b) => {
      a = ((a << 5) - a) + b.charCodeAt(0);
      return a & a;
    }, 0)) % 10000}`.padStart(5, '0');
    
    return `User ${idNumber}`;
  };

  return (
    <div className={`min-h-screen transition-colors duration-500 ${darkMode ? 'dark bg-gray-900' : 'bg-gray-100'}`}>
      <div className="max-w-2xl mx-auto p-4">
        {/* Header */}
        <header className="flex justify-between items-center mb-6 py-4 border-b dark:border-gray-700">
          <motion.h1 
            className={`text-2xl font-bold ${darkMode ? 'text-white' : 'text-gray-900'}`}
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            transition={{ duration: 0.3 }}
          >
            LHYMSS Secret
          </motion.h1>
          
          <div className="flex gap-2">
            {/* Server Message Button - only visible in developer mode */}
            {developerMode && currentView === 'feed' && (
              <Button 
                variant="ghost"
                size="icon"
                onClick={() => setShowServerMessageInput(true)}
                aria-label="Send Server Message"
                className={ghostButtonClass}
              >
                <Bell className={`h-5 w-5 ${darkMode ? 'text-white' : 'text-gray-900'}`} />
              </Button>
            )}
            
            {currentView === 'settings' && (
              <Button 
                variant="ghost"
                size="icon"
                onClick={goToHome}
                aria-label="Home"
                className={ghostButtonClass}
              >
                <Home className={`h-5 w-5 ${darkMode ? 'text-white' : 'text-gray-900'}`} />
              </Button>
            )}
            
            <Button 
              variant="ghost"
              size="icon"
              onClick={() => setCurrentView('settings')}
              aria-label="Settings"
              className={ghostButtonClass}
            >
              <Settings className={`h-5 w-5 ${darkMode ? 'text-white' : 'text-gray-900'}`} />
            </Button>
            
            <Button 
              variant="ghost"
              size="icon"
              onClick={() => setDarkMode(!darkMode)}
              aria-label={darkMode ? "Switch to light mode" : "Switch to dark mode"}
              className={ghostButtonClass}
            >
              {darkMode ? <Sun className="h-5 w-5 text-white" /> : <Moon className="h-5 w-5 text-gray-900" />}
            </Button>
          </div>
        </header>

        {/* Server Messages */}
        <AnimatePresence>
          {serverMessages.map((message) => (
            <motion.div
              key={message.id}
              initial={{ opacity: 0, y: -20 }}
              animate={{ opacity: 1, y: 0 }}
              exit={{ opacity: 0, y: -20 }}
              className={`mb-4 p-4 rounded-lg flex items-start gap-3 ${
                darkMode ? 'bg-red-900/30 border border-red-700' : 'bg-red-100 border border-red-300'
              }`}
            >
              {/* Anonymous emoji instead of AlertTriangle */}
              <div className={`flex-shrink-0 w-8 h-8 rounded-full flex items-center justify-center ${darkMode ? 'bg-gray-700' : 'bg-gray-200'}`}>
                <span className="text-lg">👤</span>
              </div>
              <div className="flex-1">
                <p className={`font-medium ${darkMode ? 'text-red-200' : 'text-red-800'}`}>
                  Server Message
                </p>
                <p className={`${darkMode ? 'text-red-300' : 'text-red-700'}`}>
                  {message.text}
                </p>
                <p className={`text-xs mt-1 ${darkMode ? 'text-red-400' : 'text-red-600'}`}>
                  {formatTime(message.timestamp)} • Will disappear in 15 seconds
                </p>
              </div>
            </motion.div>
          ))}
        </AnimatePresence>

        {/* Server Message Input Modal */}
        <AnimatePresence>
          {showServerMessageInput && (
            <motion.div
              initial={{ opacity: 0 }}
              animate={{ opacity: 1 }}
              exit={{ opacity: 0 }}
              className="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"
              onClick={() => setShowServerMessageInput(false)}
            >
              <motion.div
                initial={{ scale: 0.9, opacity: 0 }}
                animate={{ scale: 1, opacity: 1 }}
                exit={{ scale: 0.9, opacity: 0 }}
                className={`rounded-lg p-6 w-full max-w-md ${darkMode ? 'bg-gray-800' : 'bg-white'}`}
                onClick={(e) => e.stopPropagation()}
              >
                <div className="flex justify-between items-center mb-4">
                  <h3 className={`text-lg font-semibold ${darkMode ? 'text-white' : 'text-gray-900'}`}>
                    Send Server Message
                  </h3>
                  <Button 
                    variant="ghost" 
                    size="icon"
                    onClick={() => setShowServerMessageInput(false)}
                    className={ghostButtonClass}
                  >
                    <X className={`h-5 w-5 ${darkMode ? 'text-white' : 'text-gray-900'}`} />
                  </Button>
                </div>
                
                <Textarea
                  value={newServerMessage}
                  onChange={(e) => setNewServerMessage(e.target.value)}
                  placeholder="Enter server message..."
                  className={`mb-4 ${darkMode ? 'bg-gray-700 text-white border-gray-600' : ''}`}
                  rows={3}
                />
                
                <div className="flex justify-end gap-2">
                  <Button 
                    variant="outline"
                    onClick={() => setShowServerMessageInput(false)}
                    className={settingsOutlineButtonClass}
                  >
                    Cancel
                  </Button>
                  <Button 
                    onClick={handleServerMessageSubmit}
                    disabled={!newServerMessage.trim()}
                    className={settingsButtonClass}
                  >
                    Send Message
                  </Button>
                </div>
              </motion.div>
            </motion.div>
          )}
        </AnimatePresence>

        <AnimatePresence mode="wait">
          {currentView === 'feed' ? (
            <motion.div
              key="feed"
              initial={{ opacity: 0, y: 20 }}
              animate={{ opacity: 1, y: 0 }}
              exit={{ opacity: 0, y: -20 }}
              transition={{ duration: 0.3 }}
            >
              {/* Create Post */}
              <Card className={`mb-6 ${darkMode ? 'bg-gray-800 border-gray-700' : 'bg-white'}`}>
                <CardHeader>
                  <div className="flex items-center gap-3">
                    <Avatar>
                      {avatar ? (
                        <AvatarImage src={avatar} />
                      ) : (
                        <AvatarFallback className={darkMode ? 'bg-gray-700 text-white' : ''}>
                          {username.charAt(0).toUpperCase()}
                        </AvatarFallback>
                      )}
                    </Avatar>
                    <div>
                      <h2 className={`text-lg font-semibold ${darkMode ? 'text-white' : 'text-gray-900'}`}>
                        {username} <span className={`text-sm ${darkMode ? 'text-gray-400' : 'text-gray-500'}`}>{userId}</span>
                      </h2>
                    </div>
                  </div>
                </CardHeader>
                <CardContent>
                  <Textarea
                    placeholder="What's on your mind?"
                    value={newPostContent}
                    onChange={(e) => setNewPostContent(e.target.value)}
                    className={`mb-3 ${darkMode ? 'bg-gray-700 text-white border-gray-600' : ''}`}
                  />
                  
                  {/* Image Preview */}
                  {newPostImagePreview && (
                    <div className="relative mb-3">
                      <img 
                        src={newPostImagePreview} 
                        alt="Preview" 
                        className="rounded-lg max-h-60 w-full object-cover"
                      />
                      <Button
                        variant="ghost"
                        size="icon"
                        onClick={removeImage}
                        className={`absolute top-2 right-2 rounded-full ${darkMode ? 'bg-gray-800/70 hover:bg-gray-700' : 'bg-white/70 hover:bg-gray-100'}`}
                      >
                        <XCircle className={`h-6 w-6 ${darkMode ? 'text-white' : 'text-gray-900'}`} />
                      </Button>
                    </div>
                  )}
                  
                  {/* Image Upload Button */}
                  <div className="flex items-center gap-2">
                    <Button
                      variant="outline"
                      onClick={triggerImageInput}
                      className={
                        darkMode 
                          ? "bg-blue-600 text-white hover:bg-blue-700 border-blue-600 flex items-center gap-2" 
                          : "border-gray-300 text-gray-700 hover:bg-gray-100 flex items-center gap-2"
                      }
                    >
                      <Image className="h-4 w-4" />
                      Add Photo
                    </Button>
                    <input
                      type="file"
                      ref={imageInputRef}
                      className="hidden"
                      accept="image/*"
                      onChange={handleImageChange}
                    />
                    {newPostImagePreview && (
                      <Button
                        onClick={handleImageUpload}
                        className={`text-xs ${buttonClass}`}
                        size="sm"
                      >
                        Confirm Image
                      </Button>
                    )}
                  </div>
                </CardContent>
                <CardFooter className="flex justify-end">
                  <Button 
                    onClick={handlePostSubmit}
                    disabled={newPostContent.trim() === '' && !newPostImage}
                    className={buttonClass}
                  >
                    <Send className="mr-2 h-4 w-4" /> Post
                  </Button>
                </CardFooter>
              </Card>

              {/* Posts Feed */}
              <div className="space-y-6">
                {posts.map((post) => (
                  <Card 
                    key={post.id} 
                    className={`overflow-hidden relative ${darkMode ? 'bg-gray-800 border-gray-700' : 'bg-white'}`}
                  >
                    {/* Three dots menu */}
                    <div className="absolute top-4 right-4">
                      <Button
                        variant="ghost"
                        size="icon"
                        onClick={(e) => {
                          e.stopPropagation();
                          setMenuOpen(menuOpen === post.id ? null : post.id);
                        }}
                        className={ghostButtonClass}
                      >
                        <MoreHorizontal className={`h-5 w-5 ${darkMode ? 'text-white' : 'text-gray-900'}`} />
                      </Button>
                      
                      {/* Dropdown menu */}
                      <AnimatePresence>
                        {menuOpen === post.id && (
                          <motion.div
                            initial={{ opacity: 0, scale: 0.9 }}
                            animate={{ opacity: 1, scale: 1 }}
                            exit={{ opacity: 0, scale: 0.9 }}
                            className={`absolute right-0 mt-1 w-48 rounded-md shadow-lg z-10 ${
                              darkMode ? 'bg-gray-800 border border-gray-700' : 'bg-white border border-gray-200'
                            }`}
                          >
                            {/* User's own post options */}
                            {post.isUserPost && (
                              <Button
                                variant="ghost"
                                className={`w-full justify-start px-4 py-2 text-sm ${
                                  darkMode 
                                    ? 'text-red-400 hover:bg-blue-600/20 hover:text-red-300' 
                                    : 'text-red-600 hover:bg-black/10 hover:text-red-700'
                                }`}
                                onClick={() => handleDeletePost(post.id)}
                              >
                                <Trash className="h-4 w-4 mr-2" />
                                Delete Post
                              </Button>
                            )}
                            
                            {/* Developer mode options for other users' posts */}
                            {developerMode && !post.isUserPost && (
                              <>
                                <Button
                                  variant="ghost"
                                  className={`w-full justify-start px-4 py-2 text-sm ${
                                    darkMode 
                                      ? 'text-red-400 hover:bg-blue-600/20 hover:text-red-300' 
                                      : 'text-red-600 hover:bg-black/10 hover:text-red-700'
                                  }`}
                                  onClick={() => handleDeletePost(post.id)}
                                >
                                  <Trash className="h-4 w-4 mr-2" />
                                  Delete Post
                                </Button>
                                <Button
                                  variant="ghost"
                                  className={`w-full justify-start px-4 py-2 text-sm ${
                                    darkMode 
                                      ? 'text-orange-400 hover:bg-blue-600/20 hover:text-orange-300' 
                                      : 'text-orange-600 hover:bg-black/10 hover:text-orange-700'
                                  }`}
                                  onClick={() => handleBanUser(post.userId)}
                                >
                                  <Ban className="h-4 w-4 mr-2" />
                                  Ban User
                                </Button>
                              </>
                            )}
                          </motion.div>
                        )}
                      </AnimatePresence>
                    </div>
                    
                    <CardHeader className="flex flex-row items-center gap-3">
                      <Avatar>
                        {avatar && post.isUserPost ? (
                          <AvatarImage src={avatar} />
                        ) : (
                          <AvatarFallback className={darkMode ? 'bg-gray-700 text-white' : ''}>
                            {post.isUserPost ? username.charAt(0).toUpperCase() : 'U'}
                          </AvatarFallback>
                        )}
                      </Avatar>
                      <div>
                        <h3 className={`font-semibold ${darkMode ? 'text-white' : 'text-gray-900'}`}>
                          {getUsernameDisplay(post.userId, post.isUserPost, true)}
                        </h3>
                        <p className={`text-sm ${darkMode ? 'text-gray-400' : 'text-gray-500'}`}>{formatTime(post.timestamp)}</p>
                      </div>
                    </CardHeader>
                    
                    <CardContent>
                      {post.imageUrl && (
                        <div className="mb-3">
                          <img 
                            src={post.imageUrl} 
                            alt="Post" 
                            className="rounded-lg w-full max-h-96 object-cover"
                          />
                        </div>
                      )}
                      <p className={`whitespace-pre-wrap ${darkMode ? 'text-white' : 'text-gray-900'}`}>{post.content}</p>
                    </CardContent>
                    
                    <CardFooter className={`flex justify-between border-t px-6 py-3 ${darkMode ? 'border-gray-700' : 'border-gray-200'}`}>
                      <Button 
                        variant="ghost" 
                        size="sm"
                        onClick={() => handleLike(post.id)}
                        className={`flex items-center gap-1 ${post.liked ? 'text-red-500' : ghostButtonClass}`}
                      >
                        <Heart className={`h-4 w-4 ${post.liked ? 'fill-current' : ''}`} />
                        <span>{post.likes}</span>
                      </Button>
                      
                      <Button 
                        variant="ghost" 
                        size="sm"
                        onClick={() => toggleComments(post.id)}
                        className={ghostButtonClass}
                      >
                        <MessageCircle className="h-4 w-4" />
                        <span>{post.comments.length}</span>
                      </Button>
                    </CardFooter>
                    
                    {/* Comments Section */}
                    {post.showComments && (
                      <div className={`border-t px-6 py-3 ${darkMode ? 'border-gray-700' : 'border-gray-200'}`}>
                        <div className="space-y-4 mb-4">
                          {post.comments.map((comment) => (
                            <div key={comment.id} className="flex gap-3 relative">
                              {/* Developer mode comment menu */}
                              {developerMode && (
                                <div className="absolute top-0 right-0">
                                  <Button
                                    variant="ghost"
                                    size="icon"
                                    onClick={(e) => {
                                      e.stopPropagation();
                                      setCommentMenuOpen(commentMenuOpen === comment.id ? null : comment.id);
                                    }}
                                    className={`rounded-full h-6 w-6 ${ghostButtonClass}`}
                                  >
                                    <MoreHorizontal className={`h-4 w-4 ${darkMode ? 'text-white' : 'text-gray-900'}`} />
                                  </Button>
                                  
                                  {/* Comment dropdown menu */}
                                  <AnimatePresence>
                                    {commentMenuOpen === comment.id && (
                                      <motion.div
                                        initial={{ opacity: 0, scale: 0.9 }}
                                        animate={{ opacity: 1, scale: 1 }}
                                        exit={{ opacity: 0, scale: 0.9 }}
                                        className={`absolute right-0 mt-1 w-40 rounded-md shadow-lg z-10 ${
                                          darkMode ? 'bg-gray-800 border border-gray-700' : 'bg-white border border-gray-200'
                                        }`}
                                      >
                                        <Button
                                          variant="ghost"
                                          className={`w-full justify-start px-3 py-1.5 text-xs ${
                                            darkMode 
                                              ? 'text-red-400 hover:bg-blue-600/20 hover:text-red-300' 
                                              : 'text-red-600 hover:bg-black/10 hover:text-red-700'
                                          }`}
                                          onClick={() => handleDeleteComment(post.id, comment.id)}
                                        >
                                          <Trash className="h-3 w-3 mr-1" />
                                          Delete Comment
                                        </Button>
                                      </motion.div>
                                    )}
                                  </AnimatePresence>
                                </div>
                              )}
                              
                              <Avatar className="w-8 h-8">
                                <AvatarFallback className={darkMode ? 'bg-gray-700 text-white' : ''}>C</AvatarFallback>
                              </Avatar>
                              <div className="flex-1">
                                <div className="flex items-center gap-2 mb-1">
                                  <span className={`text-sm font-medium ${darkMode ? 'text-white' : 'text-gray-900'}`}>
                                    {getUsernameDisplay(comment.userId, comment.userId === 'currentUser', false)}
                                  </span>
                                </div>
                                <div className={`rounded-lg p-3 ${darkMode ? 'bg-gray-700' : 'bg-gray-100'}`}>
                                  <p className={`text-sm ${darkMode ? 'text-white' : 'text-gray-900'}`}>{comment.text}</p>
                                </div>
                                <p className={`text-xs mt-1 ${darkMode ? 'text-gray-400' : 'text-gray-500'}`}>
                                  {formatTime(comment.timestamp)}
                                </p>
                              </div>
                            </div>
                          ))}
                        </div>
                        
                        <div className="flex gap-2">
                          <Textarea
                            placeholder="Write a comment..."
                            value={newComment[post.id] || ''}
                            onChange={(e) => handleCommentChange(post.id, e.target.value)}
                            className={`flex-1 text-sm ${darkMode ? 'bg-gray-700 text-white border-gray-600' : ''}`}
                            rows={1}
                          />
                          <Button 
                            size="sm"
                            onClick={() => handleAddComment(post.id)}
                            disabled={!newComment[post.id]?.trim()}
                            className={buttonClass}
                          >
                            <Send className="h-4 w-4" />
                          </Button>
                        </div>
                      </div>
                    )}
                  </Card>
                ))}
              </div>
            </motion.div>
          ) : (
            <motion.div
              key="settings"
              initial={{ opacity: 0, y: 20 }}
              animate={{ opacity: 1, y: 0 }}
              exit={{ opacity: 0, y: -20 }}
              transition={{ duration: 0.3 }}
            >
              {/* Settings Page */}
              <Card className={`mb-6 ${darkMode ? 'bg-gray-800 border-gray-700' : 'bg-white'}`}>
                <CardHeader>
                  <h2 className={`text-xl font-bold flex items-center gap-2 ${darkMode ? 'text-white' : 'text-gray-900'}`}>
                    <Settings className="h-5 w-5" /> Settings
                  </h2>
                </CardHeader>
                <CardContent className="space-y-6">
                  {/* Profile Section */}
                  <div>
                    <h3 className={`text-lg font-semibold mb-3 ${darkMode ? 'text-white' : 'text-gray-900'}`}>Profile</h3>
                    <div className="flex items-center gap-4">
                      <div className="relative">
                        <Avatar className="h-16 w-16">
                          {avatar ? (
                            <AvatarImage src={avatar} />
                          ) : (
                            <AvatarFallback className={`text-xl ${darkMode ? 'bg-gray-700 text-white' : ''}`}>
                              {username.charAt(0).toUpperCase()}
                            </AvatarFallback>
                          )}
                        </Avatar>
                        <Button
                          size="sm"
                          className={`absolute -bottom-2 -right-2 rounded-full h-8 w-8 ${
                            darkMode ? 'bg-blue-600 hover:bg-blue-700' : 'bg-black hover:bg-gray-800'
                          } text-white`}
                          onClick={triggerFileInput}
                        >
                          <User className="h-4 w-4" />
                        </Button>
                        <input
                          type="file"
                          ref={fileInputRef}
                          className="hidden"
                          accept="image/*"
                          onChange={handleAvatarChange}
                        />
                      </div>
                      <div className="flex-1">
                        <label className={`block text-sm font-medium mb-1 ${darkMode ? 'text-gray-300' : 'text-gray-700'}`}>
                          Username
                        </label>
                        <Input
                          value={newUsername}
                          onChange={(e) => setNewUsername(e.target.value)}
                          className={darkMode ? 'bg-gray-700 text-white border-gray-600' : ''}
                        />
                        <p className={`text-xs mt-1 ${darkMode ? 'text-gray-400' : 'text-gray-500'}`}>
                          Your ID: <span className="font-mono">{userId}</span>
                        </p>
                      </div>
                    </div>
                  </div>
                  
                  {/* Developer Mode Section */}
                  <div>
                    <h3 className={`text-lg font-semibold mb-3 ${darkMode ? 'text-white' : 'text-gray-900'}`}>Developer Mode</h3>
                    {developerMode ? (
                      <div className="space-y-4">
                        <div className="flex items-center justify-between p-4 rounded-lg bg-green-500/20 dark:bg-green-500/10 border border-green-500/30">
                          <div>
                            <p className={`font-medium ${darkMode ? 'text-green-400' : 'text-green-700'}`}>Developer Mode Active</p>
                            <p className={`text-sm ${darkMode ? 'text-green-300' : 'text-green-600'}`}>
                              You have admin privileges
                            </p>
                          </div>
                          <Button 
                            variant="outline" 
                            size="sm"
                            onClick={() => setDeveloperMode(false)}
                            className={settingsOutlineButtonClass}
                          >
                            Disable
                          </Button>
                        </div>
                      </div>
                    ) : (
                      <div className="space-y-4">
                        {showPasscodeInput ? (
                          <div className="p-4 rounded-lg bg-muted dark:bg-gray-700">
                            <div className="flex justify-between items-center mb-2">
                              <label className={`font-medium ${darkMode ? 'text-white' : 'text-gray-900'}`}>
                                Enter Passcode
                              </label>
                              <Button 
                                variant="ghost" 
                                size="sm" 
                                onClick={() => setShowPasscodeInput(false)}
                                className={`p-0 h-auto ${ghostButtonClass}`}
                              >
                                <X className="h-4 w-4" />
                              </Button>
                            </div>
                            <Input
                              type="password"
                              value={passcode}
                              onChange={(e) => setPasscode(e.target.value)}
                              placeholder="Enter 4-digit passcode"
                              className={`mb-2 ${darkMode ? 'bg-gray-800 text-white border-gray-600' : ''}`}
                              maxLength={4}
                            />
                            <Button 
                              onClick={handlePasscodeSubmit}
                              className={settingsButtonClass}
                              disabled={passcode.length !== 4}
                            >
                              <Shield className="h-4 w-4 mr-2" />
                              Activate Developer Mode
                            </Button>
                          </div>
                        ) : (
                          <div className="flex items-center justify-between p-4 rounded-lg bg-muted dark:bg-gray-700">
                            <div>
                              <p className={`font-medium ${darkMode ? 'text-white' : 'text-gray-900'}`}>Enable Developer Mode</p>
                              <p className={`text-sm ${darkMode ? 'text-gray-400' : 'text-gray-500'}`}>
                                Gain admin privileges for moderation
                              </p>
                            </div>
                            <Button 
                              variant="outline" 
                              size="sm"
                              onClick={() => setShowPasscodeInput(true)}
                              className={settingsOutlineButtonClass}
                            >
                              Enable
                            </Button>
                          </div>
                        )}
                      </div>
                    )}
                  </div>
                  
                  {/* Appearance Section */}
                  <div>
                    <h3 className={`text-lg font-semibold mb-3 ${darkMode ? 'text-white' : 'text-gray-900'}`}>Appearance</h3>
                    <div className="flex items-center justify-between p-4 rounded-lg bg-muted dark:bg-gray-700">
                      <div>
                        <p className={`font-medium ${darkMode ? 'text-white' : 'text-gray-900'}`}>Dark Mode</p>
                        <p className={`text-sm ${darkMode ? 'text-gray-400' : 'text-gray-500'}`}>
                          Toggle between light and dark themes
                        </p>
                      </div>
                      <Button 
                        variant="outline" 
                        size="sm"
                        onClick={() => setDarkMode(!darkMode)}
                        className={settingsOutlineButtonClass}
                      >
                        {darkMode ? 'Disable' : 'Enable'}
                      </Button>
                    </div>
                  </div>
                </CardContent>
                <CardFooter className="flex justify-between">
                  <Button 
                    variant="outline" 
                    onClick={goToHome}
                    className={settingsOutlineButtonClass}
                  >
                    <Home className="h-4 w-4 mr-2" />
                    Home
                  </Button>
                  <Button 
                    onClick={handleSaveUsername}
                    className={settingsButtonClass}
                  >
                    Save Changes
                  </Button>
                </CardFooter>
              </Card>
            </motion.div>
          )}
        </AnimatePresence>
      </div>
    </div>
  );
}
Share
Refresh

