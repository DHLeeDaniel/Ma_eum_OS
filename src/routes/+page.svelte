<script>
  import { onMount } from 'svelte';
  
  // 페이지 데이터
  const pages = {
    'getting-started': {
      title: '시작하기',
      blocks: [
        { type: 'heading', content: 'Ma-eum OS에 오신 것을 환영합니다!' },
        { type: 'text', content: '이곳은 검색이 최적화된 워크스페이스입니다. 원하는 정보를 언제든 빠르게 찾을 수 있어요.' },
        { type: 'todo', content: '첫 번째 할일을 만들어보세요', completed: false },
        { type: 'todo', content: 'Ctrl+F로 검색을 시도해보세요', completed: false },
        { type: 'text', content: '🐆 치타 AI가 도움을 드릴 준비가 되어 있습니다!' }
      ]
    },
    'project-plan': {
      title: '프로젝트 계획',
      blocks: [
        { type: 'heading', content: 'Ma-eum OS 개발 계획' },
        { type: 'text', content: '검색 최적화에 중점을 둔 워크스페이스 개발' },
        { type: 'todo', content: '한글 검색 최적화 구현', completed: true },
        { type: 'todo', content: '치타 AI 통합', completed: false },
        { type: 'todo', content: '실시간 인덱싱 시스템', completed: false }
      ]
    },
    'ideas': {
      title: '아이디어 노트',
      blocks: [
        { type: 'heading', content: '혁신적인 아이디어들' },
        { type: 'text', content: '💡 초성 검색으로 한글 검색 속도 향상' },
        { type: 'text', content: '💡 AI가 검색 의도를 파악해서 관련 문서 추천' },
        { type: 'text', content: '💡 시간 기반 검색으로 "어제 작성한" 같은 자연어 쿼리 처리' }
      ]
    }
  };

  let currentPage = 'getting-started';
  let searchQuery = '';
  let searchResults = [];
  let showSearchPreview = false;
  let showCheetahModal = false;

  // 한글 초성 추출 함수
  function extractChosung(text) {
    return text.split('').map(char => {
      const code = char.charCodeAt(0);
      if (code >= 44032 && code <= 55203) {
        const chosung = Math.floor((code - 44032) / 588);
        return String.fromCharCode(4352 + chosung);
      }
      return char;
    }).join('');
  }

  // 검색 함수
  function performSearch(query) {
    if (!query.trim()) {
      showSearchPreview = false;
      return;
    }

    const results = [];
    const chosungQuery = extractChosung(query);
    
    Object.keys(pages).forEach(pageId => {
      const page = pages[pageId];
      
      page.blocks.forEach((block, blockIndex) => {
        const blockText = block.content.toLowerCase();
        const blockChosung = extractChosung(block.content);
        
        if (blockText.includes(query.toLowerCase()) || 
            blockChosung.includes(chosungQuery) ||
            page.title.toLowerCase().includes(query.toLowerCase())) {
          
          results.push({
            pageId,
            pageTitle: page.title,
            blockIndex,
            blockContent: block.content,
            blockType: block.type
          });
        }
      });
    });

    searchResults = results.slice(0, 5);
    showSearchPreview = results.length > 0;
  }

  // 키보드 단축키
  function handleKeydown(event) {
    if ((event.ctrlKey || event.metaKey) && event.key === 'f') {
      event.preventDefault();
      document.getElementById('search-input')?.focus();
    }
    if (event.key === 'Escape') {
      showSearchPreview = false;
      showCheetahModal = false;
    }
  }

  onMount(() => {
    document.addEventListener('keydown', handleKeydown);
    return () => {
      document.removeEventListener('keydown', handleKeydown);
    };
  });

  $: currentPageData = pages[currentPage];
</script>

<svelte:head>
  <title>Ma-eum OS - 검색 최적화 워크스페이스</title>
</svelte:head>

<div style="display: flex; height: 100vh; background: #f9fafb;">
  <!-- 사이드바 -->
  <div style="width: 256px; background: white; border-right: 1px solid #e5e7eb; display: flex; flex-direction: column;">
    <!-- 로고 & 검색 -->
    <div style="padding: 16px; border-bottom: 1px solid #f3f4f6;">
      <div style="display: flex; align-items: center; gap: 8px; margin-bottom: 16px;">
        <div style="width: 32px; height: 32px; background: linear-gradient(135deg, #f59e0b, #f97316); border-radius: 8px; display: flex; align-items: center; justify-content: center;">
          <span style="color: white; font-weight: bold; font-size: 14px;">🐆</span>
        </div>
        <span style="font-weight: bold; color: #1f2937;">Ma-eum OS</span>
      </div>
      
      <!-- 슈퍼 검색바 -->
      <div style="position: relative;">
        <input 
          id="search-input"
          type="text" 
          bind:value={searchQuery}
          on:input={() => performSearch(searchQuery)}
          placeholder="무엇을 찾고 계신가요? (Ctrl+F)"
          style="width: 100%; padding: 8px 12px; border: 1px solid #d1d5db; border-radius: 8px; font-size: 14px; outline: none;"
        >
        <div style="position: absolute; right: 8px; top: 8px;">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#9ca3af" stroke-width="2">
            <circle cx="11" cy="11" r="8"></circle>
            <path d="21 21l-4.35-4.35"></path>
          </svg>
        </div>
      </div>
      
      <!-- 검색 결과 미리보기 -->
      {#if showSearchPreview}
        <div style="position: absolute; z-index: 50; margin-top: 8px; width: 320px; background: white; border: 1px solid #d1d5db; border-radius: 8px; box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1); padding: 16px;">
          <div style="font-size: 14px; font-weight: 500; color: #374151; margin-bottom: 8px;">검색 결과</div>
          <div style="display: flex; flex-direction: column; gap: 8px;">
            {#each searchResults as result}
              <div 
                style="padding: 8px; border-radius: 4px; cursor: pointer; transition: background-color 0.2s;"
                on:click={() => {
                  currentPage = result.pageId;
                  showSearchPreview = false;
                  searchQuery = '';
                }}
                on:mouseenter={(e) => e.target.style.backgroundColor = '#f9fafb'}
                on:mouseleave={(e) => e.target.style.backgroundColor = 'transparent'}
              >
                <div style="font-size: 12px; color: #6b7280; margin-bottom: 4px;">{result.pageTitle}</div>
                <div style="font-size: 14px; color: #111827;">{result.blockContent}</div>
              </div>
            {/each}
          </div>
        </div>
      {/if}
    </div>
    
    <!-- 치타 AI 패널 -->
    <div style="padding: 16px; border-bottom: 1px solid #f3f4f6;">
      <div style="display: flex; align-items: center; gap: 8px; margin-bottom: 12px;">
        <span style="font-size: 18px;">🐆</span>
        <span style="font-weight: 500; color: #374151;">치타 AI</span>
      </div>
      
      <div style="display: flex; flex-direction: column; gap: 8px;">
        <button 
          style="width: 100%; text-align: left; font-size: 14px; color: #6b7280; background: none; border: none; cursor: pointer; transition: color 0.2s;"
          on:click={() => showCheetahModal = true}
          on:mouseenter={(e) => e.target.style.color = '#f59e0b'}
          on:mouseleave={(e) => e.target.style.color = '#6b7280'}
        >
          💡 "최근 문서 찾기"
        </button>
        <button 
          style="width: 100%; text-align: left; font-size: 14px; color: #6b7280; background: none; border: none; cursor: pointer; transition: color 0.2s;"
          on:click={() => showCheetahModal = true}
          on:mouseenter={(e) => e.target.style.color = '#f59e0b'}
          on:mouseleave={(e) => e.target.style.color = '#6b7280'}
        >
          📅 "오늘 할일 정리"
        </button>
        <button 
          style="width: 100%; text-align: left; font-size: 14px; color: #6b7280; background: none; border: none; cursor: pointer; transition: color 0.2s;"
          on:click={() => showCheetahModal = true}
          on:mouseenter={(e) => e.target.style.color = '#f59e0b'}
          on:mouseleave={(e) => e.target.style.color = '#6b7280'}
        >
          🔗 "관련 페이지 찾기"
        </button>
      </div>
    </div>
    
    <!-- 페이지 목록 -->
    <div style="flex: 1; padding: 16px;">
      <div style="font-size: 14px; font-weight: 500; color: #374151; margin-bottom: 12px;">페이지</div>
      <div style="display: flex; flex-direction: column; gap: 4px;">
        {#each Object.entries(pages) as [pageId, page]}
          <div 
            style="display: flex; align-items: center; gap: 8px; padding: 8px; border-radius: 4px; cursor: pointer; font-size: 14px; transition: background-color 0.2s; {currentPage === pageId ? 'background: #fef3c7; color: #92400e;' : 'color: #374151;'}"
            on:click={() => currentPage = pageId}
            on:mouseenter={(e) => {if (currentPage !== pageId) e.target.style.backgroundColor = '#f9fafb'}}
            on:mouseleave={(e) => {if (currentPage !== pageId) e.target.style.backgroundColor = 'transparent'}}
          >
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              {#if pageId === 'getting-started'}
                <path d="14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                <polyline points="14,2 14,8 20,8"></polyline>
                <line x1="16" y1="13" x2="8" y2="13"></line>
                <line x1="16" y1="17" x2="8" y2="17"></line>
                <polyline points="10,9 9,9 8,9"></polyline>
              {:else if pageId === 'project-plan'}
                <path d="22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z"></path>
              {:else}
                <circle cx="12" cy="12" r="3"></circle>
                <path d="M12 1v6m0 6v6m11-7h-6m-6 0H1"></path>
              {/if}
            </svg>
            <span>{page.title}</span>
          </div>
        {/each}
      </div>
    </div>
  </div>
  
  <!-- 메인 에디터 -->
  <div style="flex: 1; display: flex; flex-direction: column;">
    <!-- 상단 툴바 -->
    <div style="background: white; border-bottom: 1px solid #e5e7eb; padding: 12px 24px; display: flex; align-items: center; justify-content: space-between;">
      <div style="display: flex; align-items: center; gap: 16px;">
        <h1 style="font-size: 20px; font-weight: 600; color: #1f2937; margin: 0;">{currentPageData.title}</h1>
        <div style="display: flex; align-items: center; gap: 8px; font-size: 14px; color: #6b7280;">
          <span>마지막 수정:</span>
          <span>방금 전</span>
        </div>
      </div>
      
      <div style="display: flex; align-items: center; gap: 8px;">
        <button style="padding: 6px 12px; font-size: 14px; background: #f59e0b; color: white; border: none; border-radius: 4px; cursor: pointer; transition: background-color 0.2s;" 
                on:mouseenter={(e) => e.target.style.backgroundColor = '#d97706'}
                on:mouseleave={(e) => e.target.style.backgroundColor = '#f59e0b'}>
          공유
        </button>
        <button style="padding: 6px; color: #9ca3af; background: none; border: none; cursor: pointer;">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="12" cy="12" r="1"></circle>
            <circle cx="19" cy="12" r="1"></circle>
            <circle cx="5" cy="12" r="1"></circle>
          </svg>
        </button>
      </div>
    </div>
    
    <!-- 에디터 영역 -->
    <div style="flex: 1; overflow: auto;">
      <div style="max-width: 1024px; margin: 0 auto; padding: 32px 24px;">
        <div style="display: flex; flex-direction: column; gap: 8px;">
          {#each currentPageData.blocks as block}
            <div style="padding: 8px; border-radius: 8px; cursor: pointer; transition: background-color 0.2s;"
                 on:mouseenter={(e) => e.target.style.backgroundColor = '#fef3c7'}
                 on:mouseleave={(e) => e.target.style.backgroundColor = 'transparent'}>
              {#if block.type === 'heading'}
                <h2 style="font-size: 32px; font-weight: bold; color: #1f2937; margin: 0 0 8px 0;">{block.content}</h2>
              {:else if block.type === 'text'}
                <p style="color: #374151; line-height: 1.6; margin: 0;">{block.content}</p>
              {:else if block.type === 'todo'}
                <div style="display: flex; align-items: center; gap: 8px;">
                  <input 
                    type="checkbox" 
                    checked={block.completed}
                    style="border-radius: 4px; border: 1px solid #d1d5db;"
                  >
                  <span style="{block.completed ? 'text-decoration: line-through; color: #6b7280;' : 'color: #374151;'}">
                    {block.content}
                  </span>
                </div>
              {/if}
            </div>
          {/each}
        </div>
        
        <!-- 블록 추가 버튼 -->
        <div style="margin-top: 16px;">
          <button style="display: flex; align-items: center; gap: 8px; color: #9ca3af; background: none; border: none; cursor: pointer; font-size: 14px; transition: color 0.2s;"
                  on:mouseenter={(e) => e.target.style.color = '#6b7280'}
                  on:mouseleave={(e) => e.target.style.color = '#9ca3af'}>
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="12" y1="5" x2="12" y2="19"></line>
              <line x1="5" y1="12" x2="19" y2="12"></line>
            </svg>
            <span>블록 추가하기 (/ 명령어 사용 가능)</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</div>

<!-- 치타 AI 채팅 모달 -->
{#if showCheetahModal}
  <div style="position: fixed; inset: 0; background: rgba(0, 0, 0, 0.5); z-index: 50; display: flex; align-items: center; justify-content: center;">
    <div style="background: white; border-radius: 8px; width: 384px; height: 384px; display: flex; flex-direction: column;">
      <div style="padding: 16px; border-bottom: 1px solid #e5e7eb; display: flex; align-items: center; justify-content: space-between;">
        <div style="display: flex; align-items: center; gap: 8px;">
          <span style="font-size: 20px;">🐆</span>
          <span style="font-weight: 500;">치타와 대화하기</span>
        </div>
        <button on:click={() => showCheetahModal = false} style="color: #9ca3af; background: none; border: none; cursor: pointer;">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <line x1="18" y1="6" x2="6" y2="18"></line>
            <line x1="6" y1="6" x2="18" y2="18"></line>
          </svg>
        </button>
      </div>
      
      <div style="flex: 1; padding: 16px; overflow: auto;">
        <div style="font-size: 14px; color: #6b7280; margin-bottom: 16px;">
          <span style="font-weight: 500;">🐆 치타:</span> 안녕하세요! 무엇을 도와드릴까요?
        </div>
      </div>
      
      <div style="padding: 16px; border-top: 1px solid #e5e7eb;">
        <div style="display: flex; gap: 8px;">
          <input 
            type="text" 
            placeholder="치타에게 질문하기..."
            style="flex: 1; padding: 8px 12px; border: 1px solid #d1d5db; border-radius: 4px; font-size: 14px; outline: none;"
          >
<!-- svelte-ignore a11y_consider_explicit_label -->
<button style="padding: 8px 16px; background: #f59e0b; color: white; border: none; border-radius: 4px; cursor: pointer; transition: background-color 0.2s;">
  <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
    <line x1="22" y1="2" x2="11" y2="13"></line>
    <polygon points="22,2 15,22 11,13 2,9 22,2"></polygon>
  </svg>
</button>
        </div>
      </div>
    </div>
  </div>
{/if}