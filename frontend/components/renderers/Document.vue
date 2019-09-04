<template>
    <div class="document-renderer">
        <header>
            <h1>{{title}}</h1>
            <button @click="render = !render">{{render ? 'render' : 'edit'}}</button>
        </header>
        <main>
            <textarea v-model="content" ref="textarea" :class="render ? 'hide' : ''"></textarea>
            <div v-html="renderedContent" class="renderbox"></div>
        </main>
    </div>
</template>

<script>
import md from 'markdown-it'
import _ from 'lodash'
export default {
    data() {
        return {
            lastCursor: null,
            title: "Document Title",
            content:
`# Testing

## This could be a subtitle!

and this is standard text

\`\`\`
const tellAll = () => {
    console.log(' code blocks make me sad ')
}
\`\`\`

that's all for now
`,
            render: false
        }
    },
    computed: {
        renderedContent() {
            return md().render(this.content);
        }
    },
    methods: {
        update: _.debounce(function (e) {
            this.content = e.target.value
        }, 300),
        handleClick(e) {
            console.log(e)
            console.log("target", e.target)
            this.$refs.textarea.focus()
        }
    },
    mounted() {
        var source = document.querySelector('.renderbox')
        source.onclick = replaceAndSet;
        var v_this = this;

        function replaceAndSet (e) {
            for (const span of source.querySelectorAll('.cursor--span')) {
                span.parentNode.removeChild(span)
            }

            var selectionRange;
            if (window.getSelection) {
                selectionRange = window.getSelection().getRangeAt(0);
            }
            
            var newInput = v_this.$refs.textarea
            
            var parent = selectionRange.startContainer.parentElement;
            var textLines = parent.childNodes;
            var offset = 0, i = 0, curr = null;
            for(; i < source.childElementCount; i++){
                curr = source.children[i];
                // compensate for ::before's
                switch (curr.tagName) {
                    case 'H1':
                        offset+=2
                        break;
                    case 'H2':
                        offset+=3
                        break;
                    case 'PRE':
                        if(parent.tagName == 'CODE') {
                            offset -=  (18 + curr.textContent.length)
                            break;
                        } offset += 7;
                        break;
                    case 'H3':
                        offset+=4
                        break;
                    case 'H4':
                        offset+=5
                        break;
                    default:
                        break;
                }
                if(curr === parent) break; 
                else offset += curr.textContent.length;
                offset+=2;
            }

            const clickChar = getCursorLoc(textLines, selectionRange);
            const initInnerHTML = parent.textContent;
            const beforeText = initInnerHTML.substring(0, clickChar)
            const afterText = initInnerHTML.substring(clickChar)            
            parent.innerHTML = `<span>${beforeText}</span><span class="cursor--span">|</span><span>${afterText}</span>`

            const cursor = document.createElement(parent.tagName)
            const boundingRect = document.querySelector('.cursor--span').getBoundingClientRect();
            cursor.innerHTML = "|";
            cursor.classList.add('cursor--span')
            cursor.style.position = 'fixed';
            cursor.style.left = `${boundingRect.left}px`
            cursor.style.top = `${boundingRect.y}px`
            parent.innerHTML = initInnerHTML;
            source.appendChild(cursor)

            newInput.focus()
            newInput.setSelectionRange(clickChar + offset, clickChar + offset)
        }

        function getCursorLoc (nodes, range) {
            var prevChars = 0;
            var clickContent = range.startContainer.textContent;
            var clickLine;
            var node;
            for (var x = nodes.length - 1; x >= 0; x--) {
                node = nodes[x];
                if (clickContent == node.textContent) clickLine = x;
                else if (clickLine && (node.nodeName == "#text")) prevChars += node.textContent.length + 1;
            }
            
            return range.startOffset + prevChars;
        }

    }
}
</script>

<style lang="postcss" scoped>
.document-renderer {
    overflow-wrap: break-word;
    @apply w-full h-full text-gray-200;
}
header h1 {
    @apply text-2xl my-4;
}
main {
    min-height: 100px;
    @apply rounded border-2 border-gray-700 px-6 pb-6 pt-3 relative;
}
textarea {
    background: none;
    min-height: 40vh;
    transition: all 0.3s ease-out;
    opacity: 1;
    resize: none;
    @apply w-full h-full;
}
.hide {
    opacity: 0;
    position: absolute;
    z-index: -1;
}
.renderbox {
    @apply pt-3;
}
.renderbox >>> h1 {
    @apply text-2xl my-4;
}
.renderbox >>> h2 {
    @apply text-xl my-3;
}
.renderbox >>> h3 {
    @apply text-xl my-2;
}
.renderbox >>> h4 {
    @apply text-lg my-1;
}
.renderbox >>> p {
    @apply text-base;
}
.renderbox >>> pre {
    @apply text-base my-2 p-4 bg-gray-800;
}

.renderbox >>> h1::before, 
.renderbox >>> h2::before,
.renderbox >>> h3::before,
.renderbox >>> h4::before,
.renderbox >>> pre::before,
.renderbox >>> li::before {
    text-align: right;
    position: absolute;
    left: -1rem;
    transform: translateX(-100%)
}
.renderbox >>> h1::before {
    content: "#";
}
.renderbox >>> h2::before {
    content: "##";
}
.renderbox >>> h3::before {
    content: "###";
}
.renderbox >>> h4::before {
    content: "####";
}
.renderbox >>> pre::before {
    content: "```"
}
.renderbox >>> li::before {
    content: "-"
}

.renderbox >>> h1:first-of-type {
    @apply text-2xl mb-4 -mt-4;
}
.renderbox >>> .cursor--span {
    margin: 0;
    margin-left: -2px;
    animation: blink-animation 1s steps(5, start) infinite;
    @apply text-orange-200 inline-block;
}

.renderbox >>> .cursor--span::before {
    content: none;
}

@keyframes blink-animation {
  to {
    visibility: hidden;
  }
}
</style>